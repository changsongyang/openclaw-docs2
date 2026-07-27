---
read_when:
    - Refactorización del ciclo de vida de las sesiones ACP o de la limpieza de procesos ACPX
    - Depuración de procesos huérfanos de ACPX, reutilización de PID o seguridad de limpieza en entornos con varios Gateway
    - Cambiar la visibilidad de sessions_list para sesiones de ACP o de subagentes generadas
    - Diseño de metadatos de propiedad para tareas en segundo plano, sesiones ACP o concesiones de procesos
sidebarTitle: ACP lifecycle refactor
summary: Plan de migración para explicitar la propiedad de la sesión de ACP y del proceso de ACPX
title: Refactorización del ciclo de vida de ACP
x-i18n:
    generated_at: "2026-07-26T05:54:32Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bda66f0acc93216c3d9386ca3ebf7f544efd306cd7f53386391f0c48e5dc8f06
    source_path: refactor/acp.md
    workflow: 16
---

Actualmente, el ciclo de vida de ACP funciona, pero gran parte de él se infiere a posteriori.
La limpieza de procesos reconstruye la propiedad a partir de PID, cadenas de comandos, rutas
de envoltorios y la tabla de procesos activa. La visibilidad de las sesiones reconstruye la propiedad
a partir de cadenas de claves de sesión y consultas secundarias de `sessions.list({ spawnedBy })`.
Esto permite aplicar correcciones específicas, pero también facilita pasar por alto casos límite:
la reutilización de PID, los comandos entrecomillados, los procesos descendientes de adaptadores, las raíces de estado
de varios Gateway, `cancel` frente a `close` y la visibilidad de `tree` frente a `all` se convierten en lugares
independientes donde volver a descubrir las mismas reglas de propiedad.

Esta refactorización convierte la propiedad en un concepto de primera clase. El objetivo no es una nueva superficie
de producto ACP, sino un contrato interno más seguro para el comportamiento existente de ACP y ACPX.

## Objetivos

- La limpieza nunca envía una señal a un proceso salvo que las pruebas activas actuales coincidan con un
  arrendamiento propiedad de OpenClaw.
- `cancel`, `close` y la recolección al inicio tienen intenciones de ciclo de vida distintas.
- `sessions_list`, `sessions_history`, `sessions_send` y las comprobaciones de estado utilizan
  el mismo modelo de sesión propiedad del solicitante.
- Las instalaciones con varios Gateway no pueden recolectar los envoltorios ACPX de otras instalaciones.
- Los registros antiguos de sesiones ACPX siguen funcionando durante la migración.
- El entorno de ejecución sigue siendo propiedad del plugin; el núcleo no conoce los detalles del paquete ACPX.

## No son objetivos

- Sustituir ACPX o cambiar la superficie pública del comando `/acp`.
- Trasladar al núcleo el comportamiento de adaptadores ACP específico de proveedores.
- Exigir a los usuarios que limpien manualmente el estado antes de actualizar.
- Hacer que `cancel` cierre sesiones ACP reutilizables.

## Modelo objetivo

### Identidad de instancia del Gateway

Cada proceso de Gateway debe tener un id. estable de instancia del entorno de ejecución:

```ts
type GatewayInstanceId = string;
```

Puede generarse al iniciar el Gateway y conservarse en el estado durante la vida útil de
esa instalación. No es un secreto de seguridad; es un discriminador de propiedad utilizado
para evitar confundir los procesos ACP de un Gateway con los procesos de otro Gateway.

### Propiedad de sesiones ACP

Cada sesión ACP iniciada debe tener metadatos de propiedad normalizados:

```ts
type AcpSessionOwner = {
  sessionKey: string;
  spawnedBy?: string;
  parentSessionKey?: string;
  ownerSessionKey: string;
  agentId: string;
  backend: "acpx";
  gatewayInstanceId: GatewayInstanceId;
  createdAt: number;
};
```

El Gateway debe devolver estos campos en las filas de sesión donde se conozcan.
El filtrado de visibilidad debe ser una comprobación pura de los metadatos de la fila:

```ts
canSeeSessionRow({
  row,
  requesterSessionKey,
  visibility,
  a2aPolicy,
});
```

Esto elimina las llamadas secundarias ocultas a `sessions.list({ spawnedBy })` de
las comprobaciones de visibilidad. Un proceso secundario ACP iniciado entre agentes pertenece al solicitante porque
así lo indica la fila, no porque una segunda consulta consiga encontrarlo.

### Arrendamientos de procesos ACPX

Cada inicio de un envoltorio generado debe crear un registro de arrendamiento:

```ts
type AcpxProcessLease = {
  leaseId: string;
  gatewayInstanceId: GatewayInstanceId;
  sessionKey: string;
  wrapperRoot: string;
  wrapperPath: string;
  rootPid: number;
  processGroupId?: number;
  commandHash: string;
  startedAt: number;
  state: "open" | "closing" | "closed" | "lost";
};
```

El proceso del envoltorio recibe el id. de arrendamiento y el id. de instancia del Gateway como
argumentos portables:

```sh
--openclaw-acpx-lease-id ... --openclaw-gateway-instance-id ...
```

Cuando la plataforma lo permita, la verificación debe preferir metadatos del proceso activo
que no puedan confundirse por el entrecomillado del comando:

- el PID raíz todavía existe
- la ruta activa del envoltorio está bajo `wrapperRoot`
- el grupo de procesos coincide con el arrendamiento cuando está disponible
- los argumentos contienen el id. de arrendamiento esperado
- el hash del comando o la ruta del ejecutable coincide con el arrendamiento

Si no se puede verificar el proceso activo, la limpieza adopta un comportamiento de cierre seguro.

## Controlador del ciclo de vida

Introduzca un único controlador del ciclo de vida de ACPX que posea los arrendamientos de procesos y la política
de limpieza:

```ts
interface AcpxLifecycleController {
  ensureSession(input: AcpRuntimeEnsureInput): Promise<AcpRuntimeHandle>;
  cancelTurn(handle: AcpRuntimeHandle): Promise<void>;
  closeSession(input: {
    handle: AcpRuntimeHandle;
    discardPersistentState?: boolean;
    reason?: string;
  }): Promise<void>;
  reapStartupOrphans(): Promise<void>;
  verifyOwnedTree(lease: AcpxProcessLease): Promise<OwnedProcessTree | null>;
}
```

`cancelTurn` solo solicita la cancelación del turno. No debe recolectar procesos de envoltorio
o adaptador reutilizables.

`closeSession` puede realizar la recolección, pero solo después de cargar el registro de sesión,
cargar el arrendamiento y verificar que el árbol de procesos activo todavía pertenece a ese
arrendamiento.

`reapStartupOrphans` parte de los arrendamientos abiertos del estado. Puede utilizar la tabla
de procesos para encontrar descendientes, pero no debe examinar primero comandos arbitrarios
que parezcan de ACP y decidir después que probablemente son nuestros.

## Contrato del envoltorio

Los envoltorios generados deben seguir siendo pequeños. Deben:

- iniciar el adaptador en un grupo de procesos cuando se admita
- reenviar las señales normales de terminación al grupo de procesos
- detectar la muerte del proceso padre
- cuando muera el proceso padre, enviar SIGTERM y mantener activo el envoltorio hasta que se ejecute
  el recurso alternativo SIGKILL
- comunicar el PID raíz y el id. del grupo de procesos al controlador del ciclo de vida cuando
  estén disponibles

Los envoltorios no deben decidir la política de sesiones. Solo aplican la limpieza local del árbol
de procesos para su propio grupo de adaptadores.

## Contrato de visibilidad de sesiones

La visibilidad debe utilizar la propiedad normalizada de las filas:

```ts
type SessionVisibilityInput = {
  requesterSessionKey: string;
  row: {
    key: string;
    agentId: string;
    ownerSessionKey?: string;
    spawnedBy?: string;
    parentSessionKey?: string;
  };
  visibility: "self" | "tree" | "agent" | "all";
  a2aPolicy: AgentToAgentPolicy;
};
```

Reglas:

- `self`: solo la sesión del solicitante.
- `tree`: la sesión del solicitante más las filas que pertenecen al solicitante o se iniciaron desde ella.
- `all`: todas las filas del mismo agente, las filas entre agentes permitidas por a2a y las filas
  iniciadas entre agentes que pertenecen al solicitante, incluso cuando a2a general está desactivado.
- `agent`: solo el mismo agente, salvo que una relación explícita de propiedad indique que la fila
  pertenece al solicitante.

Esto hace que `tree` y `all` sean monótonos: `all` no debe ocultar un proceso secundario propio que
`tree` mostraría.

## Plan de migración

### Fase 1: añadir identidad y arrendamientos

- Añadir `gatewayInstanceId` al estado del Gateway.
- Añadir un almacén de arrendamientos ACPX bajo el directorio de estado de ACPX.
- Escribir un arrendamiento antes de iniciar un envoltorio generado.
- Guardar `leaseId` en los nuevos registros de sesiones ACPX.
- Conservar los campos existentes de PID y comando para los registros antiguos.

### Fase 2: limpieza basada primero en arrendamientos

- Cambiar la limpieza de cierre para que cargue primero `leaseId`.
- Verificar la propiedad del proceso activo con el arrendamiento antes de enviar señales.
- Conservar el recurso alternativo actual del PID raíz y de la raíz de envoltorios solo para registros heredados.
- Marcar los arrendamientos como `closed` después de una limpieza verificada.
- Marcar los arrendamientos como `lost` cuando el proceso haya desaparecido antes de la limpieza.

### Fase 3: recolección al inicio basada primero en arrendamientos

- La recolección al inicio examina los arrendamientos abiertos.
- Para cada arrendamiento, verificar el proceso raíz y recopilar los descendientes.
- Recolectar los árboles verificados empezando por los procesos secundarios.
- Caducar los arrendamientos antiguos `closed` y `lost` con un periodo de retención limitado.
- Conservar el examen de marcadores de comandos solo como recurso alternativo heredado temporal, protegido por
  la raíz de envoltorios y la instancia del Gateway cuando sea posible.

### Fase 4: filas de propiedad de sesiones

- Añadir metadatos de propiedad a las filas de sesión del Gateway.
- Hacer que los escritores de ACPX, subagentes, tareas en segundo plano y almacenes de sesiones rellenen
  `ownerSessionKey` o `spawnedBy`.
- Convertir las comprobaciones de visibilidad de sesiones para que utilicen los metadatos de las filas.
- Eliminar las consultas secundarias a `sessions.list({ spawnedBy })` durante la comprobación de visibilidad.

### Fase 5: eliminar heurísticas heredadas

Después de un ciclo de lanzamiento:

- dejar de depender de las cadenas de comandos raíz almacenadas para la limpieza de ACPX no heredada
- eliminar los exámenes de marcadores de comandos al inicio
- eliminar las consultas alternativas de listas para la visibilidad
- mantener un comportamiento defensivo de cierre seguro para arrendamientos ausentes o no verificables

## Pruebas

Añada dos conjuntos de pruebas basados en tablas.

Simulador del ciclo de vida de procesos:

- PID reutilizado por un proceso no relacionado
- PID reutilizado por la raíz de envoltorios de otro Gateway
- el comando de envoltorio almacenado está entrecomillado para el shell, pero el comando `ps` activo no
- el proceso secundario del adaptador termina, pero un proceso descendiente permanece en el grupo de procesos
- el recurso alternativo SIGTERM al morir el proceso padre llega a SIGKILL
- el listado de procesos no está disponible
- arrendamiento obsoleto con un proceso ausente
- proceso huérfano al inicio con envoltorio, proceso secundario del adaptador y proceso descendiente

Matriz de visibilidad de sesiones:

- `self`, `tree`, `agent`, `all`
- a2a activado y desactivado
- fila del mismo agente
- fila entre agentes
- fila ACP entre agentes iniciada y propiedad del solicitante
- solicitante en entorno aislado limitado a `tree`
- acciones de lista, historial, envío y estado

La invariante importante: un proceso secundario iniciado que pertenece al solicitante es visible siempre que
la visibilidad configurada incluya el árbol de sesiones del solicitante, y `all` no es
menos capaz que `tree`.

## Notas de compatibilidad

Es posible que los registros antiguos de sesiones no tengan `leaseId`. Deben utilizar la ruta de limpieza
heredada con cierre seguro:

- exigir un proceso raíz activo
- exigir la propiedad de la raíz de envoltorios cuando se espere un envoltorio generado
- exigir la concordancia del comando para raíces sin envoltorio
- no enviar nunca señales basándose únicamente en metadatos obsoletos del PID almacenado

Si no se puede verificar un registro heredado, no se debe modificar. La limpieza de arrendamientos al inicio y
el siguiente ciclo de lanzamiento deberían retirar finalmente el recurso alternativo.

## Criterios de éxito

- Cerrar una sesión ACPX antigua u obsoleta no puede terminar el proceso de otro Gateway.
- La muerte del proceso padre no deja en ejecución procesos descendientes persistentes del adaptador.
- `cancel` cancela el turno activo sin cerrar las sesiones reutilizables.
- `sessions_list` puede mostrar procesos secundarios ACP entre agentes que pertenecen al solicitante tanto con
  `tree` como con `all`.
- La limpieza al inicio se basa en arrendamientos, no en exámenes amplios de cadenas de comandos.
- Las pruebas específicas de las matrices de procesos y visibilidad abarcan todos los casos límite que
  anteriormente requerían correcciones puntuales durante la revisión.
