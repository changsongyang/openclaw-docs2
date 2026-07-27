---
x-i18n:
    generated_at: "2026-07-26T04:41:57Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarms — distribución en abanico de agentes y orquestación en modo de código

Estado: publicado — sustituido por `docs/tools/swarm.md`. Este documento se conserva como
registro del diseño de implementación.

## 1. Qué es y por qué

Un **swarm** consta de muchos subagentes orquestados de forma determinista desde un script
en modo de código: distribuir N lectores en abanico, verificar los hallazgos de forma adversarial, sintetizarlos mediante un
priorizador con estado y repetir según las puertas de decisión. El flujo de control (`Promise.all`,
`while`, `if`) _es_ la orquestación; deliberadamente **no hay ningún DSL de grafos,
ningún modo nuevo ni ninguna superficie nueva de herramientas de nivel superior**.

El modo de código de OpenClaw (QuickJS-WASI, instantánea/reanudación, solicitudes de puente) es el
sustrato. Una llamada de puente aparcada sobrevive a una instantánea de la VM, al reinicio del Gateway y
se reanuda exactamente donde se detuvo; es más robusto que los diseños de reproducción de diarios y
no impone restricciones de determinismo a los scripts.

Nomenclatura: el nombre en el producto y la documentación es **Swarm**. Los identificadores de código se mantienen literales:
API invitada `agents.*`, configuración `tools.swarm`, columnas de grupo `swarm`.

## 2. Decisiones (mantenedor, 2026-07-17)

- Coste: límites de configuración aplicados; presupuesto de tokens por swarm opcional. No hay ningún presupuesto obligatorio.
- Aprobaciones: los procesos secundarios se ejecutan **con cierre en caso de error y sin interacción**. Las acciones que requieren
  aprobación se deniegan; la denegación se incluye en el resultado del proceso secundario; el script
  decide. La distribución en abanico no genera una avalancha de solicitudes al operador.
- La v1 solo admite scripts ad hoc escritos por el modelo. Flujos de trabajo guardados o con nombre y entrada
  mediante CLI/Cron: más adelante (el modo de código sin interfaz ya existe para Cron).
- Identidad del proceso secundario: agente trabajador dedicado de forma predeterminada mediante la configuración `tools.swarm.defaultAgentId`
  (validada con la lista existente de destinos de subagentes permitidos); se puede sustituir en cada creación mediante
  `agentId`. El núcleo no incluye ningún id de agente integrado; la documentación recomienda una configuración
  de agente `worker` ligera.
- No hay cambios en el código fuente de Codex. El arnés de Codex utiliza el patrón de creación/espera (§8).

## 3. Descripción general de la arquitectura

```
script en modo de código (VM QuickJS, gateway)   script V8 de Codex (proceso de Codex)
  agents.run(...) ── llamada de puente aparcada    tools.sessions_spawn / tools.agents_wait
        │                                                │ RPC de elemento/herramienta/llamada (≤600s cada una)
        ▼                                                ▼
             NÚCLEO (independiente del arnés, este repositorio)
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  registro de subagentes (SQLite): registros de finalización del recopilador, id del grupo de swarm
        │
  procesos secundarios = sesiones ordinarias de subagentes (con límite de carril, aprobaciones con cierre en caso de error)
        │
  SSE sessions.changed ──► puntos de la interfaz de control / barra lateral / mensaje de estado del canal
```

Un único propietario canónico de la semántica de creación/finalización/resolución (herramientas del núcleo + registro).
Dos transportes de espera: QuickJS aparca indefinidamente una llamada de puente (instantánea);
Codex consulta `agents_wait` mediante RPC limitadas.

## 4. Puerta de configuración (v1)

Nuevo `tools.swarm` (global + sustitución por agente, con el mismo patrón de combinación que
`tools.codeMode`):

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // puerta principal, DESACTIVADA de forma predeterminada
    "maxConcurrent": 8,          // procesos secundarios ejecutados simultáneamente (límite del carril de swarm)
    "maxChildrenPerGroup": 50,   // procesos secundarios activos por grupo de swarm
    "maxTotalPerGroup": 200,     // número de creaciones durante la vida útil por grupo (protección contra descontrol)
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // opcional; id del agente secundario cuando la creación omite agentId
  }
}
```

- Zod: unión `boolean | strict object` como `CodeModeSchema`
  (`src/config/zod-schema.agent-runtime.ts`); `swarm: true` → `{enabled: true}`.
- Tipos en `src/config/types.tools.ts` (tanto por agente como en el nivel superior `tools`),
  etiquetas en `schema.labels.ts`, ayuda en `schema.help.runtime.ts`.
- Función auxiliar de resolución `resolveSwarmConfig(cfg, agentId)` que refleja
  `resolveCodeModeConfig` (`src/agents/code-mode.ts:215`) y restringe todos los números.
- Efectos de la puerta cuando está desactivada: la herramienta `agents_wait` no aparece en los catálogos;
  los parámetros `collect`/`outputSchema`/`fastMode`/`groupId` de `sessions_spawn`
  se rechazan con un error claro que indica la clave de configuración. No cambia ningún otro comportamiento.
- `defaultAgentId` se valida mediante `resolveSubagentAllowedTargetIds`
  (`src/agents/subagent-target-policy.ts`); id desconocido → error de creación, sin alternativa.

## 5. Núcleo: creación en modo recopilador + `agents_wait` (v1)

### 5.1 Incorporaciones de `sessions_spawn` (todas condicionadas a que swarm esté activado)

- `collect: boolean`: cuando es verdadero, la ejecución secundaria se registra con
  `expectsCompletionMessage: false` y un **registro de finalización del recopilador**
  en lugar de la entrega de anuncios o instrucciones. La herramienta devuelve `{ runId, sessionKey }`
  inmediatamente. Sin vinculación a canales o hilos.
- `outputSchema: object`: esquema JSON. Se añade una herramienta sintética
  `structured_output` a la superficie de herramientas del proceso secundario; una adición al prompt del sistema
  le indica que la llame exactamente una vez con el resultado final. Si falla la validación,
  el proceso secundario recibe un recordatorio para volver a intentarlo una vez; después, el registro de finalización
  contiene `structured: undefined`, además del texto sin procesar y un `schemaError`.
- `fastMode: true | "auto" | false`: se transmite al parche de sesión del proceso secundario
  junto con el modelo y el razonamiento mediante `resolveSubagentModelAndThinkingPlan`
  (`src/agents/subagent-spawn-plan.ts`), utilizando el eje existente `FastMode`
  (`src/shared/fast-mode.ts`). Si se omite, se hereda.
- `groupId: string`: marca del grupo de swarm. El valor predeterminado es
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>`. Se conserva en el
  registro y en la fila de sesión del proceso secundario. Se utiliza para los límites, los listados, el archivado
  por lotes y los puntos.
- `label: string` ya existe; aparece en los puntos y en `subagents list`.
- Id del agente secundario: `params.agentId` → de lo contrario `tools.swarm.defaultAgentId` → de lo contrario
  el agente solicitante (comportamiento existente).

### 5.2 Aprobaciones con cierre en caso de error

Los procesos secundarios recopiladores se ejecutan con un contexto de aprobación no interactivo: cualquier llamada a una herramienta
que requiera la aprobación del operador se resuelve como una denegación estructurada
(`approval_required`) visible para el proceso secundario, del que se espera que informe del
bloqueo en su resultado. Implementación: reutilizar el mecanismo existente de políticas de aprobación
de ejecución/herramientas con un resolutor `deny` forzado para las ejecuciones secundarias en modo recopilador.
No se emiten eventos de aprobación a las superficies del operador desde los procesos secundarios recopiladores.

### 5.3 Herramienta `agents_wait` (nueva, condicionada)

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- Devuelve el resultado en cuanto se completa **al menos un** id (semántica de primera finalización/carrera,
  permite pipelines) o cuando vence el tiempo de espera con `completed: []`.
- El valor predeterminado de `timeoutSeconds` es 30, restringido a `waitTimeoutSecondsMax`.
- Idempotente: los ids ya completados vuelven a devolver sus registros (los registros se
  conservan hasta que se archiva el grupo). Id desconocido → entrada de error por id, no una excepción.
- Propiedad: solo la sesión que creó una ejecución (o su cadena de antecesores) puede esperarla;
  es la misma regla de propiedad que `wait` en el modo de código (`code-mode.ts:1684`).
- Registro: los registros de finalización se almacenan en el registro SQLite de subagentes existente
  (`subagent-registry.store.sqlite.ts`): campos nuevos, ningún almacén nuevo y
  ningún incremento de la versión del esquema (solo columnas aditivas; consulte la restricción del §9).

### 5.4 Aplicación de límites

- `maxConcurrent`: los procesos secundarios recopiladores se ejecutan en el carril de subagentes existente, pero
  se cuentan por grupo de swarm; las creaciones que superan el límite se ponen en cola FIFO (en el host, dentro de la
  ruta de creación; se devuelve runId inmediatamente y la ejecución comienza cuando se libera una plaza).
- `maxChildrenPerGroup` / `maxTotalPerGroup`: la creación se rechaza con un error tipado
  cuando se supera el límite; el texto del error indica la clave de configuración.
- Profundidad: los procesos secundarios recopiladores conservan la semántica de `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH`
  (los procesos secundarios son hojas salvo que se configure explícitamente el anidamiento).

## 6. Contrato de pruebas (v1, carril A)

- Pruebas unitarias: resolución y restricción de la configuración; rechazos de la puerta cuando está desactivada; valor predeterminado de groupId;
  aplicación de límites (poner en cola + rechazar); semántica de carrera de la espera; idempotencia de la espera;
  denegación de propiedad; validación de la salida estructurada + recordatorio para reintentar +
  ruta de schemaError; transmisión de fastMode al parche de sesión; validación de defaultAgentId.
- Integración (vitest, entorno de ejecución del modelo simulado): crear 3 procesos secundarios recopiladores, esperar
  en un bucle, comprobar el orden de la primera finalización y el vaciado final; simulación de reinicio
  del Gateway: recarga del registro → la espera se resuelve a partir de la finalización conservada.
- Todas las pruebas están ubicadas junto a `*.test.ts`; no hay llamadas a modelos en vivo.

## 7. Superficie invitada de QuickJS (carril B, después del núcleo)

- Globales invitados instalados en `CONTROLLER_SOURCE`
  (`src/agents/code-mode.worker.ts:190-374`), nombres reservados añadidos en
  `code-mode-namespaces.ts`:
  - `agents.run(prompt, opts) → Promise<result|structured>`: función de conveniencia:
    creación del recopilador + espera aparcada en un método de puente dedicado (`agentWait`)
    que el host resuelve al finalizar (sin sondeo; compatible con instantáneas).
  - `agents.session(system, opts) → Promise<handle>`;
    `handle.send(input, opts) → Promise<...>`; `handle.close()`. (v1.1:
    se publica después de run(); utiliza `mode:"session"` + registros de recopilador por turno).
  - `phase(title)`, `log(message)`: notificaciones de puente sin espera de respuesta →
    eventos de progreso del swarm.
- Métodos de puente añadidos a `CodeModeBridgeMethod` (`code-mode.ts:91`):
  `agentSpawn`, `agentWait`, `swarmNote`. `agentSpawn`/`agentWait` son
  seguros para la reproducción **por construcción**: clave de idempotencia `(codeModeRunId, bridgeId)`
  almacenada en el registro; el reinicio vuelve a resolver a partir de las finalizaciones conservadas
  y nunca crea duplicados.
- Las llamadas de puente `agentWait` pendientes amplían el TTL de la instantánea de la ejecución (el conjunto
  de agentes pendientes es la señal; no hay ninguna marca).
- El archivo virtual `API.read("agents.d.ts")` documenta la superficie tipada y los
  patrones de distribución en abanico/puerta/ciclo (`createCodeModeApiVirtualFiles`,
  `code-mode-namespaces.ts:876`).

## 8. Proyección del arnés de Codex (carril posterior)

- `sessions_spawn` (con los parámetros nuevos) y `agents_wait` pasan por el
  puente existente de herramientas dinámicas; dentro de los scripts en modo de código de Codex aparecen
  automáticamente como `tools.*` (verificado: `codex-rs/code-mode/src/runtime/globals.rs:14-65`,
  `codex-rs/core/src/tools/spec_plan.rs:448-507`).
- `agents_wait` recibe la clase de tiempo de espera largo para herramientas dinámicas (límite de 600s;
  `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`) y se
  marca como seguro frente a tiempos de espera y reproducciones.
- Clave de grupo para procesos principales de Codex: `swarm:<parentSessionKey>:<turnId>`.
- Los subagentes `spawn_agent` nativos de Codex coexisten; sus filas de réplica de tareas alimentan
  la misma superficie de progreso.

## 9. Persistencia y retención

- No hay almacenes nuevos. Los registros amplían las tablas SQLite del registro de subagentes existente;
  los procesos secundarios son filas `sessions` ordinarias. Solo columnas aditivas:
  **cualquier cambio que requiera incrementar la versión del esquema de SQLite necesita primero
  la aprobación explícita del mantenedor** (política del repositorio).
- Id del grupo de swarm en el registro + metadatos de sesión del proceso secundario.
- Retención: los registros de recopiladores completados sobreviven hasta el **archivado del grupo**:
  cuando finaliza la ejecución principal (o vence el TTL), los procesos secundarios del grupo se archivan
  por lotes (ampliar el barrido existente `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`
  para que opere por grupo).

## 10. Superficie de progreso («los puntos») — carril posterior

- Implícita y controlada por el arnés. Derivada del SSE `sessions.changed` existente +
  el registro; las notas `phase`/`log` añaden semántica. Sin representación controlada por agentes.
- Interfaz de control: representador `swarm` en la familia de widgets del espacio de trabajo
  (`ui/src/lib/workspace/widgets/`): cuadrícula de puntos agrupada por fase, línea del
  narrador, estado/etiqueta/modelo por punto; el árbol de procesos secundarios de la barra lateral no cambia.
- Canales: un mensaje de estado editado y limitado por frecuencia por grupo (seguir
  `docs/concepts/streaming.md`; nunca mensajes por proceso secundario).

## 11. Página Labs (interfaz de control, vía independiente)

Settings → **Labs**: controles de activación de funciones experimentales, con **Code Mode**
y **Swarm** como primeras entradas. Cada fila: nombre, descripción de una línea, enlace a la documentación y control conectado
mediante el RPC `config.patch` existente (parche de combinación RFC 7396: establecer
`tools.codeMode.enabled` / `tools.swarm.enabled`), además de una indicación de «reinicio necesario»
cuando corresponda. Es fácil de encontrar, pero el texto deja claro su estado experimental.
i18n: todas las cadenas pasan por `en.ts` y el pipeline de sincronización habituales.

## 12. Ubicación (más adelante)

- `placement` permite elegir al generar: `"local"` (predeterminado) | `"cloud:<profile>"` mediante
  el envío existente al entorno de trabajo (`sessions.dispatch`); ubicación agrupada
  más adelante si los procesos secundarios en el entorno aislado SSH de la máquina compartida resultan insuficientes.
- La máquina virtual del orquestador siempre permanece en el gateway; la estabilización, los puntos y el presupuesto son
  independientes de la ubicación.

## 13. Objetivos excluidos

- Sin DSL de grafos: el flujo de control es el grafo (deliberado y documentado).
- Sin cambios en el código fuente de Codex; sin reutilizar los componentes internos de Code Mode de Codex.
- Sin flujos de trabajo guardados o con nombre en v1; sin punto de entrada de CLI.
- Sin propagación al nivel superior de las aprobaciones del operador de cada proceso secundario.
- Sin aprovisionamiento en la nube de 1:1 a escala de distribución en abanico.
- Sin capas de compatibilidad en el entorno de ejecución en estado estable; Swarm es una superficie nueva y está restringida.

## 14. Fases de compilación / división de PR

1. **Vía A (núcleo)**: configuración de §4 + generación/espera/límites/aprobaciones de §5 + pruebas de §6.
2. **Vía C (página Labs)**: §11; independiente, puede integrarse primero.
3. **Vía B (superficie de QuickJS)**: §7; después de integrar los contratos de A.
4. Renderizador de puntos (§10), proyección de Codex (§8), `agents.session` (§7 v1.1),
   ubicación (§12), reescritura de la documentación de usuario: PR posteriores en ese orden.

Cada PR: Pipeline de CI en verde, `$autoreview` limpio, desactivado de forma predeterminada y rama principal lista para publicar.
