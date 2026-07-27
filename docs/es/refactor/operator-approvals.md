---
read_when:
    - Cambiar el ciclo de vida, el almacenamiento, el protocolo o la autorización de la aprobación de exec o de plugins
    - Añadir enlaces de aprobación o controles de aprobación nativos a un canal
    - Proyección de las aprobaciones de sesiones secundarias en las vistas principales o del orquestador
summary: Diseño de aprobaciones duraderas con enlaces profundos en la interfaz de control, las aplicaciones nativas, los canales y las sesiones principales
title: Aprobaciones del operador en múltiples superficies
x-i18n:
    generated_at: "2026-07-26T04:56:54Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 9defdaada1911df1184f64429e1787c4881e735c433d6dbc30a5946e11cc7cce
    source_path: refactor/operator-approvals.md
    workflow: 16
---

# Aprobaciones del operador en múltiples superficies

Este diseño realiza el seguimiento de [#103505](https://github.com/openclaw/openclaw/issues/103505). Sustituye la autoridad de aprobación local del proceso por un único ciclo de vida propiedad del Gateway y respaldado por SQLite. Cada aprobación de ejecución o de plugin/herramienta propiedad del Gateway obtiene un ID estable, una ruta autenticada de la Control UI, una resolución atómica en la que prevalece la primera respuesta y proyecciones exclusivas para operadores en los flujos de su sesión de origen y sus sesiones antecesoras.

Las acciones integradas y los enlaces profundos coexisten. No hay ningún selector de modo de aprobación.

## Objetivos

- Un objeto de aprobación duradero para las barreras de ejecución y de plugin/herramienta.
- Ruta estable `${controlUiBasePath}/approve/{approvalId}`.
- Resolución desde cualquier Control UI, aplicación nativa o superficie de canal autorizada.
- Comportamiento atómico en el que prevalece la primera respuesta entre superficies simultáneas.
- Los reintentos idénticos son idempotentes; las respuestas tardías que entren en conflicto no pueden sobrescribir la ganadora.
- Los tiempos de espera, los veredictos de confianza con formato incorrecto, las rutas ausentes, las cancelaciones y los reinicios se cierran de forma segura.
- Los eventos de solicitud y terminales llegan a la sesión de origen y a todos los propietarios principales o de orquestación pertinentes.
- Los canales reciben acciones tipadas de aprobación y navegación; los datos de devolución de llamada del transporte permanecen privados para el canal.
- Los métodos existentes del Gateway para ejecución y plugins mantienen la compatibilidad mientras su implementación converge en un único servicio.

## Fuera de los objetivos

- Persistir o reanudar la propia ejecución bloqueada de la herramienta después de reiniciar el Gateway.
- Convertir un ID o una URL de aprobación en una credencial de portador.
- Añadir solicitudes de aprobación a transcripciones visibles para el modelo o despertar a agentes principales.
- Trasladar la política de aprobación, los comandos del producto o la autorización de revisores a los plugins de canal.
- Clonar el estado de aprobación por canal, dispositivo o antecesor.
- Rediseñar las listas de permitidos de ejecución, la composición de políticas de plugins o la persistencia de `allow-always`, salvo cuando sea necesario para que los resultados terminales sean inequívocos.
- Hacer que una TUI integrada sin Gateway sea accesible de forma remota en el primer incremento. Sigue siendo exclusivamente local y debe cerrarse de forma segura cuando no haya ningún revisor.

## Línea de base previa al despliegue y mapa de evidencias

Esta tabla registra el estado de la implementación cuando se abrió #103505. Las secciones de despliegue posteriores realizan el seguimiento del registro duradero, las acciones tipadas, la página de enlace profundo y los incrementos del cliente nativo desarrollados sobre esa línea de base.

| Superficie        | Punto de entrada y propietario de referencia                                                                                                                                  | Comportamiento y carencia de referencia                                                                                                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ejecución del agente | `src/agents/bash-tools.exec-approval-request.ts`, `src/agents/bash-tools.exec-host-shared.ts`                                                                   | El registro en dos fases de `exec.approval.*` evita una condición de carrera temprana de `/approve`, pero el tiempo de espera aún puede convertirse en una autorización mediante `askFallback`.                                                        |
| Barrera de herramienta del plugin | `src/agents/agent-tools.before-tool-call.ts`                                                                                                                    | Solicita `plugin.approval.*`; `timeoutBehavior: "allow"` puede aprobar una barrera cuyo tiempo de espera ha vencido. El modo integrado tiene una autoridad local del proceso independiente en `src/infra/embedded-plugin-approval-broker.ts`. |
| Barrera de Node del plugin | `src/gateway/node-invoke-plugin-policy.ts`                                                                                                                      | Crea y difunde directamente mediante el gestor de plugins, lo que duplica parte del ciclo de vida del método del servidor.                                                                                 |
| Autoridad del Gateway | `src/gateway/server-aux-handlers.ts`, `src/gateway/exec-approval-manager.ts`, `src/gateway/server-methods/approval-shared.ts`                                   | Los gestores independientes de ejecución y plugins utilizan mapas locales del proceso. Las entradas terminales se conservan durante 15 segundos. La primera respuesta solo prevalece dentro de un proceso.                                          |
| Protocolo del Gateway | `packages/gateway-protocol/src/schema/exec-approvals.ts`, `packages/gateway-protocol/src/schema/plugin-approvals.ts`, `src/gateway/methods/core-descriptors.ts` | La ejecución tiene `get` solo para elementos pendientes; los plugins no tienen `get`; no existe ninguna consulta terminal independiente del tipo para un enlace profundo.                                                                                   |
| Entrega           | `src/infra/exec-approval-channel-runtime.ts`, `src/infra/approval-native-runtime.ts`, `src/infra/approval-handler-runtime.ts`                                   | Admite enrutamiento de origen, mensajes directos a aprobadores, reproducción de elementos pendientes, controladores nativos y limpieza terminal dentro del proceso. Un seguimiento independiente añade la conciliación terminal duradera.                          |
| Acciones portátiles | `src/interactive/payload.ts`, `src/plugin-sdk/interactive-runtime.ts`, `src/plugin-sdk/approval-reply-runtime.ts`                                               | Los botones de aprobación son acciones de comando que contienen `/approve ...`; los destinos de URL y aplicación web son campos de botón sin tipar.                                                                           |
| Telegram          | `extensions/telegram/src/approval-handler.runtime.ts`, `extensions/telegram/src/button-types.ts`                                                                | El renderizador analiza el texto del comando para reconocer la semántica de aprobación antes de generar datos privados de devolución de llamada.                                                                                     |
| Control UI        | `ui/src/app/exec-approval.ts`, `ui/src/app/overlays.ts`, `ui/src/components/exec-approval.ts`                                                                   | La interfaz de aprobación es un cuadro de diálogo modal global. `ui/src/app-route-paths.ts` y `ui/src/app-routes.ts` utilizan rutas exactas y redirigen las rutas desconocidas a Chat.                                                    |
| Propiedad de la sesión | `src/agents/subagent-registry.types.ts`, `src/agents/subagent-registry-read.ts`, `src/config/sessions/types.ts`                                                 | Existen la propiedad del controlador, del solicitante, del padre explícito y de la creación heredada, pero los eventos de aprobación no se proyectan en esos flujos de sesión.                                                    |
| Estado compartido | `src/state/openclaw-state-schema.sql`, `src/state/openclaw-state-db.ts`                                                                                         | Las transacciones inmediatas existentes y las actualizaciones condicionales de Kysely permiten una operación duradera de comparación e intercambio en `state/openclaw.sqlite`.                                                                   |

Las pruebas actuales representativas incluyen `src/gateway/exec-approval-manager.test.ts`, `src/gateway/server-methods/approval-shared.test.ts`, `src/agents/bash-tools.exec-gateway-approval.e2e.test.ts`, `extensions/telegram/src/approval-handler.runtime.test.ts` y `ui/src/e2e/approval-flow.e2e.test.ts`.

El SDK de plugins sigue siendo el único límite de canales/plugins. Los cambios en el entorno de ejecución y la presentación de aprobaciones deben exportarse mediante las subrutas existentes `src/plugin-sdk/approval-*.ts` y `src/plugin-sdk/interactive-runtime.ts`; el código de producción de los plugins no debe importar elementos internos del Gateway.

## Trabajo previo

Omnigent proporciona una experiencia de usuario y una semántica de errores útiles:

- [`approval.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/runtime/policies/approval.py) pone ASK en espera, aplica tiempos de espera por política y solo considera como aprobación una aceptación exacta.
- [`sessions.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/routes/sessions.py) contiene la barrera del entorno nativo del lado del servidor y la proyección de solicitudes y resoluciones a los antecesores.
- [`ApprovePage.tsx`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/web/src/pages/ApprovePage.tsx) proporciona la página independiente de aprobación para dispositivos móviles.

No se debe copiar de forma acrítica su afirmación sobre el almacenamiento. El estado pendiente activo actual es local del proceso en [`_elicitation_registry.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/_elicitation_registry.py), y [`e3b1f2a4c9d7_drop_pending_tool_calls_table.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/db/migrations/versions/e3b1f2a4c9d7_drop_pending_tool_calls_table.py) elimina la tabla de elementos pendientes sin utilizar. OpenClaw va deliberadamente más allá: SQLite es la autoridad y cada transición terminal es una operación de comparación e intercambio en la base de datos.

## Arquitectura y propiedad

El Gateway es propietario del ciclo de vida:

1. Un agente, enlace de plugin o política de Node proporciona una solicitud específica del tipo y una vinculación de ejecución local del proceso.
2. El Gateway la valida y crea una proyección saneada para el revisor.
3. El servicio de aprobación calcula una audiencia de origen y propietarios, inserta la fila canónica y después registra el proceso de espera dentro del proceso.
4. Tras la inserción duradera, el Gateway publica los eventos de aprobación existentes, las proyecciones de sesión, las notificaciones de canal y las notificaciones push nativas.
5. Todas las superficies resuelven mediante el mismo servicio.
6. El servicio confirma una transición terminal, reactiva el proceso de espera del entorno de ejecución y publica las proyecciones terminales.
7. Un fallo en la entrega de eventos nunca revierte la decisión confirmada; los clientes se recuperan mediante `approval.get` o la reproducción de la lista.

Límites de propiedad:

- `src/gateway/`: servicio de aprobación, autorización, adaptadores RPC, construcción de URL, ciclo de vida de los procesos de espera y publicación de eventos.
- `src/state/`: esquema compartido y tipos de Kysely generados.
- `src/infra/`: modelos de vista saneados de aprobación y construcción de presentaciones portátiles.
- `src/agents/`: solicita, espera y aplica el veredicto devuelto; sin persistencia.
- `src/channels/` y `extensions/*`: renderizan acciones tipadas, autorizan a los usuarios del canal, codifican devoluciones de llamada privadas y actualizan los controles entregados.
- `src/plugin-sdk/`: únicamente contratos públicos de aprobación y presentación.
- `ui/`: página independiente y clientes existentes de cola y cuadro de diálogo modal.

El proceso de espera dentro del proceso es un mecanismo de notificación, no una autoridad. El registro inserta la fila e instala el proceso de espera de forma síncrona antes de publicar la solicitud, por lo que ningún elemento de resolución puede intercalarse entre esos pasos. Cada elemento de resolución posterior confirma mediante SQLite antes de resolver ese proceso de espera.

## Registro persistente

Añadir una tabla `operator_approvals` a la base de datos de estado compartido.

| Columna                                            | Propósito                                                                                                                                       |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval_id`                                      | ID canónico único globalmente. Conserve los ID de ejecución existentes y los ID `plugin:` para mantener la compatibilidad del protocolo, pero nunca deduzca el tipo a partir del prefijo.      |
| `resolution_ref`                                   | Localizador base64url SHA-256 completo y único para devoluciones de llamada de transporte que no pueden contener el ID canónico. No constituye autorización ni es un ID de URL pública. |
| `kind`                                             | Discriminador `exec \| plugin` cerrado.                                                                                                        |
| `status`                                           | Estado `pending \| allowed \| denied \| expired \| cancelled` cerrado.                                                                          |
| `presentation_json`                                | Proyección del revisor validada y etiquetada por tipo. Las solicitudes sin procesar del entorno de ejecución, las vinculaciones de comandos y las cargas útiles de las devoluciones de llamada permanecen en el proceso local.               |
| `source_agent_id`, `source_session_key`            | Identidad de origen y ancla de la proyección de sesión. La clave de sesión es duradera; el UUID de sesión rotatorio no lo es.                                          |
| `audience_session_keys_json`                       | Matriz JSON ordenada y sin duplicados producida por el recorrido acotado en anchura de la propiedad. Los eventos solicitados y terminales utilizan esta misma instantánea. |
| `requested_by_device_id`, `requested_by_client_id` | Metadatos duraderos del solicitante y de auditoría. El ID de conexión permanece en memoria y no es una entidad principal entre superficies.                                         |
| `reviewer_device_ids_json`                         | Dispositivos de revisores específicos opcionales proporcionados únicamente por el entorno de ejecución de aprobación de confianza.                                                  |
| `runtime_epoch`                                    | Época del proceso propietario de la ejecución estacionada; se utiliza para cancelar filas huérfanas después de un reinicio.                                                     |
| `created_at_ms`, `expires_at_ms`, `updated_at_ms`  | Temporización autoritativa.                                                                                                                         |
| `decision`                                         | Decisión explícita del usuario, cuando exista.                                                                                                       |
| `terminal_reason`                                  | Motivo cerrado, como `user`, `timeout`, `malformed-verdict`, `no-route`, `run-aborted` o `gateway-restart`.                                |
| `resolved_at_ms`, `resolver_kind`, `resolver_id`   | Identidad del ganador y de auditoría conservada en el servidor. Las proyecciones del revisor omiten los identificadores sin procesar del resolutor.                                           |
| `consumed_at_ms`, `consumed_by`                    | Protección de repetición independiente para `allow-once`; el consumo no debe borrar la decisión registrada.                                                       |

Índices requeridos:

| Índice                                     | Propósito                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| unique `(resolution_ref)`                  | Rechazar durante la inserción la ambigüedad `approval_id`/`resolution_ref` entre columnas. |
| `(status, expires_at_ms)`                  | Buscar aprobaciones pendientes y conciliar los plazos autoritativos.               |
| `(source_session_key, created_at_ms DESC)` | Repetir las aprobaciones recientes de una sesión de origen.                             |
| `(resolved_at_ms)`                         | Purgar las aprobaciones terminales conservadas según la política fija de retención.  |

Las matrices de audiencia son pequeñas y acotadas. La repetición filtrada por sesión selecciona primero las filas pendientes visibles mediante Kysely y, a continuación, decodifica y filtra en el código de la aplicación las matrices de audiencia acotadas; no utiliza coincidencias de cadenas ni consultas JSON de SQL sin procesar.

Conserve las filas terminales durante 30 días, de acuerdo con la retención de auditoría de metadatos en `src/audit/audit-event-store.ts`. La purga es una política fija de mantenimiento, no una nueva superficie de configuración. La base de datos es un estado privado del plano de control local, pero las API de revisores nunca deben exponer la solicitud almacenada completa ni la vinculación del entorno de ejecución.

## Máquina de estados y comparación e intercambio

Solo son válidas estas transiciones:

- `pending -> allowed`: `allow-once` o `allow-always` explícito.
- `pending -> denied`: denegación explícita, veredicto terminal malformado de confianza o ausencia de ruta de entrega.
- `pending -> expired`: se alcanza el plazo autoritativo.
- `pending -> cancelled`: interrupción de la ejecución, cierre ordenado o recuperación de elementos huérfanos tras un reinicio.

Todo estado terminal no permitido tiene como veredicto efectivo la denegación.

La resolución utiliza una transacción SQLite inmediata y una actualización condicional de Kysely equivalente a:

```sql
UPDATE operator_approvals
SET status = ?, decision = ?, terminal_reason = ?, resolved_at_ms = ?
WHERE approval_id = ?
  AND status = 'pending'
  AND expires_at_ms > ?;
```

Si la actualización no afecta a ninguna fila, la misma transacción lee el registro:

- Ausente o no autorizado: devolver no encontrado; no revelar su existencia.
- Aún pendiente, pero se alcanzó el plazo: compararlo e intercambiarlo por `expired` y, a continuación, devolver esa fila terminal.
- Misma decisión registrada: devolver un resultado satisfactorio idempotente con el ganador registrado.
- Decisión diferente: la API unificada devuelve `applied: false` con el ganador registrado; los adaptadores heredados conservan `APPROVAL_ALREADY_RESOLVED` cuando así lo requiere su contrato publicado.
- Cualquier estado terminal: no modificarlo nunca.

`now == expires_at_ms` está caducado. La hora del Gateway es autoritativa.

La ejecución de `allow-once` utiliza una segunda operación CAS sobre `consumed_at_ms IS NULL`, vinculada al contexto exacto existente del comando o de la ejecución del sistema. La fila de aprobación permanece como registro de auditoría después del consumo.

La entrada HTTP/RPC malformada que no pueda autenticarse ni identificar una aprobación se rechaza sin modificaciones y nunca puede aprobar. Un veredicto terminal malformado recibido de un arnés o proceso de espera de confianza para una aprobación conocida realiza la transición a `denied`.

## API del Gateway

Añada métodos de revisión independientes del tipo:

| Método                                    | Contrato                                                                                                                                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval.get { id }`                     | Devuelve una proyección terminal pendiente visible o conservada.                                                                                                                                                          |
| `approval.resolve { id, kind, decision }` | Acepta el ID canónico o la referencia de transporte de tamaño fijo y, a continuación, ejecuta la autorización, la validación del tipo y de la decisión permitida, la conciliación del plazo y la operación CAS terminal. La respuesta siempre incluye el ID canónico. |

Después de una operación CAS satisfactoria, devuelva inmediatamente la proyección confirmada. Los eventos heredados, los reenviadores de canales y los finalizadores push son acciones posteriores de mejor esfuerzo; una superficie lenta o con errores no debe retrasar ni revertir la respuesta ganadora.

La validación de solicitudes específica de cada tipo permanece en `exec.approval.request` y `plugin.approval.request`. Los elementos existentes `exec.approval.get/list/waitDecision/resolve` y `plugin.approval.list/waitDecision/resolve` se convierten en adaptadores del límite del protocolo para el servicio canónico porque forman parte de la API publicada del Gateway. Los invocadores internos migran al servicio en el mismo cambio.

Una proyección de revisor es una unión etiquetada:

```ts
type OperatorApproval = {
  id: string;
  status: OperatorApprovalStatus;
  presentation:
    | { kind: "exec"; commandText: string /* vista previa segura de la ejecución */ }
    | { kind: "plugin"; title: string; description: string /* vista previa segura del plugin */ };
  // campos comunes del ciclo de vida
};
```

La ruta estable se deriva, no se conserva. `approval.get` devuelve `urlPath`; las superficies que conozcan un origen público aprobado también pueden recibir un `url` absoluto. Las instantáneas del revisor omiten las claves de sesión del origen y de la audiencia. El Gateway conserva esas claves de enrutamiento en el servidor para la proyección independiente `session.approval`.

## Eventos y acciones portátiles

El PR 1 conserva los nombres de eventos, las cargas útiles y los filtros de destinatarios existentes a nivel de registro ya publicados:

- `exec.approval.requested`
- `exec.approval.resolved`
- `plugin.approval.requested`
- `plugin.approval.resolved`

Esos eventos heredados pueden contener la solicitud completa del entorno de ejecución, por lo que no deben difundirse a todos los clientes asociados a aprobaciones. El PR 5 añade campos de ciclo de vida etiquetados (`status`, `sourceSessionKey`, `urlPath`, metadatos terminales y un `kind` a nivel de presentación) mediante la proyección saneada del ciclo de vida, en lugar de ampliar la entrega de eventos heredados.

Añada un evento de proyección `session.approval` asociado a aprobaciones. Publique una sola vez el evento canónico con las claves de audiencia conservadas; los suscriptores de la sesión exacta reciben el mismo evento por cada clave coincidente:

- `sessionKey`: flujo que recibe la proyección.
- `sourceSessionKey`: elemento secundario u origen que activó la puerta.
- `phase`: `pending \| terminal`, discriminado según el estado de la aprobación.
- una proyección segura de `OperatorApproval`.

Los clientes se suscriben mediante `sessions.messages.subscribe { key, agentId?, includeApprovals: true }`. La respuesta satisfactoria añade un `approvalReplay` que contiene hasta 1,000 aprobaciones pendientes actuales para esa clave de flujo exacta que el cliente suscriptor también está autorizado a revisar a nivel de registro. `truncated: false` hace que la repetición filtrada sea autoritativa y los clientes que vuelven a conectarse reemplazan su conjunto pendiente local por ella; `truncated: true` es una señal de sobrecarga y los clientes deben conservar las entradas locales aún no vistas hasta que la consulta canónica o los eventos posteriores del ciclo de vida las resuelvan. Un tiempo de espera duradero posterior detectado durante la repetición emite lápidas terminales únicamente para las audiencias suscritas y autorizadas a nivel de registro antes de devolver la nueva instantánea. `operator.admin` puede suscribirse directamente; los clientes de alcance más limitado requieren tanto una identidad de dispositivo emparejado como `operator.approvals`. La suscripción a la sesión por sí sola nunca concede visibilidad de las aprobaciones.

Registre el evento en `operator.approvals` dentro de `src/gateway/server-broadcast.ts`. La proyección es observacional: nunca añade filas a la transcripción, emite `sessions.changed` ni reactiva un agente.

Amplíe `MessagePresentationAction` en `src/interactive/payload.ts`:

```ts
type MessagePresentationAction =
  | { type: "command"; command: string }
  | { type: "callback"; value: string }
  | {
      type: "approval";
      approvalId: string;
      approvalKind: "exec" | "plugin";
      decision: ExecApprovalDecision;
    }
  | { type: "url"; url: string }
  | { type: "web-app"; url: string };
```

El núcleo crea acciones de decisión tipadas y un enlace de revisión independiente cuando hay disponible un origen absoluto aprobado de la interfaz de control. Los canales codifican una acción de aprobación en su propio formato de devolución de llamada y envían la resolución al servicio canónico. Una devolución de llamada usa el ID canónico exacto cuando cabe; de lo contrario, usa el `resolution_ref` de resumen completo único de la fila. La referencia es solo una clave compacta de búsqueda: siguen aplicándose la autenticación normal del Gateway, la autorización del registro, el tipo explícito, la validación de decisiones permitidas, la conciliación del plazo y el CAS de primera respuesta. Los canales no deben truncar los ID, resolver prefijos hash, analizar texto de `/approve` ni inferir el tipo a partir de un prefijo de ID.

Mantenga `button.url`, `button.webApp` y los controles de aprobación respaldados por comandos como entradas de compatibilidad obsoletas del SDK de plugins. Normalícelos en el límite del SDK; migre cada llamador interno incluido en el mismo PR. `/approve {id} {decision}` sigue siendo una alternativa textual y un comando de CLI/chat, no el contrato semántico del botón.

## Interfaz de control

La ruta es `${basePath}/approve/{approvalId}`. El ID es el único parámetro de ruta; la identidad de la sesión de origen procede del registro.

Como el enrutador actual tiene rutas estáticas exactas y reescribe las rutas desconocidas a Chat, detecte este enlace profundo en `ui/src/app/bootstrap.ts` antes de la normalización habitual de rutas. Reutilice la configuración normal del Gateway y de autenticación, pero represente una página de aprobación independiente fuera del contenedor de la barra lateral y del modal global.

El documento pertenece al Gateway que sirvió su URL. Su conexión inicial ignora la selección persistente del Gateway remoto de la aplicación completa sin cambiar ni copiar la configuración de esa selección; solo la autenticación permanece limitada a la sesión del Gateway servidor. La autenticación nativa de confianza o una sustitución de `gatewayUrl` confirmada por separado pueden redirigirlo. El núcleo reserva el espacio de nombres de un segmento `/approve` antes de las rutas HTTP de plugins y de la detección de extensiones estáticas, incluidos los ID que terminan en `.json` o `.js`; cuando el servicio de la interfaz de control está desactivado, la ruta reservada se cierra ante fallos con `404`. Mantenga la página en el paquete principal de la interfaz de control para que un fragmento diferido fallido no deje bloqueada una decisión de seguridad en un indicador de carga.

Estados de la página:

- cargando
- autenticación requerida
- pendiente
- resolviendo
- aprobado o denegado aquí
- resuelto en otro lugar
- expirado
- cancelado
- prohibido/no encontrado
- error de conexión con reintento

La página llama al RPC del Gateway, no a una segunda API REST sin autenticar. Al actualizar el navegador, se vuelve a leer el estado duradero. Nunca coloca las credenciales del Gateway en la URL, la consulta ni el fragmento.

## Autorización y privacidad

La URL es un localizador, no una autoridad. La resolución requiere:

1. conexión autenticada al Gateway;
2. `operator.approvals` o `operator.admin`;
3. autorización del revisor a nivel de registro.

Reglas a nivel de registro:

- `operator.admin` puede revisar.
- `reviewer_device_ids` es autoritativo cuando está presente. Solo puede revisar un dispositivo
  `operator.approvals` emparejado que figure en la lista; el dispositivo solicitante no tiene acceso
  implícito a menos que también figure en ella.
- Sin una lista explícita de revisores, el dispositivo solicitante
  `operator.approvals` emparejado puede revisar su propio registro.
- Los registros realmente heredados sin vinculación de solicitante o revisor conservan una visibilidad amplia
  para los dispositivos emparejados, de modo que las actualizaciones no dejen bloqueado el trabajo ya pendiente.
- Los entornos de ejecución internos sin dispositivo pueden resolver, pero no leer, mediante la conexión
  del entorno de ejecución de aprobación con alcance limitado. Esa autoridad procede únicamente del token
  del entorno de ejecución autenticado por el servidor; los campos públicos `approval.resolve` no pueden
  generarla.
- La propiedad de la conexión activa del solicitante sigue siendo válida para los adaptadores heredados; nunca se
  infiere a partir de un nombre de cliente coincidente.
- La pertenencia a la audiencia solo cambia la presentación. Nunca amplía la autorización.

`approval.get` expone únicamente la proyección saneada para revisores y omite las claves internas de enrutamiento de origen y audiencia. El evento `session.approval` del PR 5 transporta su único destino `sessionKey` más `sourceSessionKey` después de que el Gateway aplique en el servidor la instantánea persistente de la audiencia. Los eventos existentes de ejecución/plugins conservan su carga útil histórica y sus destinatarios restringidos hasta que los consumidores migren. La solicitud ejecutable, la vinculación del comando y la continuación permanecen únicamente en el proceso local en espera. La fila duradera contiene la presentación segura junto con metadatos de ciclo de vida, enrutamiento y auditoría; nunca almacena valores de entorno sin procesar, credenciales, encabezados de autenticación ni datos de devolución de llamada del canal.

## Proyección de audiencia

Calcule la audiencia una vez antes de la inserción y conserve la instantánea ordenada. La propiedad es un grafo, no siempre una única cadena de ascendencia: un elemento secundario puede tener tanto un controlador actual como un solicitante original, y esos propietarios pueden conducir a raíces diferentes.

Use un recorrido determinista en anchura:

1. Inicialice la cola con la clave de sesión de origen.
2. Para cada clave extraída de la cola, lea la fila más reciente del registro de subagentes y añada a la cola ambas aristas de propiedad distintas en orden fijo: `controllerSessionKey` y después `requesterSessionKey`.
3. Cuando exista una fila de registro utilizable, no siga también el linaje de entradas de sesión, que puede estar obsoleto tras una redirección. De lo contrario, añada a la cola la única arista alternativa actual `parentSessionKey ?? spawnedBy`.
4. Normalice y elimine duplicados al añadir a la cola para que prevalezca la primera ruta, que es la más corta.
5. Deténgase al alcanzar 64 claves únicas; este límite de tamaño de la audiencia también limita la profundidad del recorrido.

El origen del registro es `src/agents/subagent-registry-read.ts`; los campos de propiedad se definen en `src/agents/subagent-registry.types.ts`. Los campos alternativos de sesión se definen en `src/config/sessions/types.ts`.

Las proyecciones solicitadas y terminales usan la misma audiencia persistente, incluso si el foco o la propiedad del controlador cambian mientras la aprobación está pendiente. Esto garantiza la limpieza terminal de cada flujo de sesión de la audiencia que recibió la proyección de la solicitud. La resolución siempre se dirige al ID de aprobación de origen; las sesiones de la audiencia nunca reciben un estado de aprobación clonado. La limpieza de mensajes de canal reenviados sigue siendo el seguimiento independiente del localizador de entrega que se describe a continuación.

No escriba mensajes de transcripción, inyecte prompts del sistema, inicie turnos de propietarios ni emita `sessions.changed` únicamente para una aprobación.

## Convergencia de superficies de entrega

Los controladores de aprobación nativos ya conservan sus entradas de mensajes entregados el tiempo suficiente para sustituir o retirar los controles activos. Actualmente, los mensajes de aprobación genéricos reenviados descartan el `MessageReceipt`, por lo que una decisión tomada en otra superficie puede hacer que sus controles antiguos sigan pareciendo pendientes. Un seguimiento independiente cierra esa brecha mediante una tabla secundaria `operator_approval_deliveries` en la base de datos de estado compartida.

Cada fila almacena el ID de aprobación, un ID de entrega único, el canal, la cuenta y la ruta exacta, un localizador privado de mensajes del canal limitado y validado como JSON, las marcas de tiempo de entrega y el estado de finalización. Nunca almacena datos de devolución de llamada, tokens de decisión ni solicitudes de aprobación sin procesar. El canal posee la codificación del localizador y la mutación de mensajes; el núcleo posee el estado canónico, la selección de destinos, la política de reintentos y el texto terminal alternativo.

El registro de entrega y la resolución terminal gestionan las condiciones de carrera de forma segura:

1. Después de que un envío pendiente devuelva su recibo, inserte el localizador de entrega y lea el estado de la aprobación principal en una sola transacción.
2. Si la aprobación principal ya está en estado terminal, programe la finalización inmediata en lugar de dejar pendiente la entrega tardía.
3. Cada transición terminal confirmada programa por separado todas las filas de entrega sin finalizar; las difusiones descartables no son el desencadenante.
4. Un finalizador de canal informa `replaced`, `retired` o `unsupported`. La sustitución suprime un mensaje terminal duplicado; la retirada envía el seguimiento terminal existente; la falta de compatibilidad o un fallo recurre a la alternativa sin revertir el CAS de aprobación.
5. Al iniciarse, se reintentan las aprobaciones terminales con entregas sin finalizar, lo que hace que la limpieza sea resistente a reinicios del Gateway.

Este ciclo de vida del transporte es un enlace opcional del adaptador de entrega, no un representador ni una acción de mensaje orientada al modelo. Los mensajes C2C/grupales de QQ no disponen actualmente de una API para editar, eliminar o borrar el teclado; ese adaptador sigue sin ser compatible y solo puede mostrar la verdad canónica después de un clic posterior hasta que el transporte obtenga una API de mutación.

## Semántica de reinicio, tiempo de espera y rutas

La persistencia en SQLite no implica la reanudación de la ejecución. Las vinculaciones de comandos y herramientas permanecen en memoria porque pueden contener datos confidenciales del entorno de ejecución y no constituyen un contrato de trabajo reanudable.

Al iniciarse el Gateway:

- genere una nueva época del entorno de ejecución;
- realice una transición atómica de las filas pendientes de épocas anteriores a `cancelled` con el motivo `gateway-restart`;
- conserve las filas para que sus URL expliquen lo sucedido;
- nunca ejecute una aprobación posterior contra una vinculación de entorno de ejecución inexistente.

Los temporizadores son optimizaciones de activación. La autoridad del plazo se almacena en `expires_at_ms`; las lecturas, esperas y resoluciones ejecutan siempre la conciliación de la expiración.

Comportamiento estricto final:

- tiempo de espera -> `expired`, denegar;
- sin ruta -> `denied`, denegar;
- cancelación de ejecución -> `cancelled`, denegar;
- veredicto de confianza con formato incorrecto -> `denied`, denegar;
- solo una decisión explícita permitida de autorización -> `allowed`.

El comportamiento de ejecución distribuido actualmente sigue entrando en conflicto con este contrato:

- `src/agents/bash-tools.exec-host-shared.ts` puede aplicar `askFallback`.
- `docs/tools/exec-approvals.md` y `docs/cli/approvals.md` documentan esa superficie.

Las aprobaciones de plugins ahora se cierran ante fallos por tiempo de espera y veredictos con formato incorrecto; el campo heredado
`timeoutBehavior` sigue aceptándose, pero se ignora. El seguimiento de semántica estricta
de ejecución debe actualizar conjuntamente el código, los tipos, la documentación, las pruebas y el registro de cambios, con
una revisión explícita de los propietarios y de seguridad. `askFallback` puede seguir describiendo
la selección de políticas previa a la puerta durante la migración, pero no debe convertir en aprobación
el tiempo de espera de un registro pendiente creado.

## Plan de compatibilidad

- Protocolo del Gateway aditivo; sin incremento de la versión del protocolo.
- Conserve los métodos y eventos existentes de ejecución/plugins en el límite externo.
- Mantenga los ID existentes, incluidos los prefijos `plugin:`, pero deje de usar los prefijos como información de tipo.
- Mantenga el comportamiento del comando de texto `/approve`.
- Mantenga los campos heredados de URL/Web App de botones y las acciones de comandos como entrada de compatibilidad del SDK de plugins; la nueva salida del núcleo es tipada.
- Migre todos los canales incluidos y los llamadores internos en el mismo cambio de acciones tipadas.
- Añada una entrada al registro de cambios para la nueva URL/página y para el cambio posterior del comportamiento del tiempo de espera.
- No añada una configuración de modo de elicitación.

## Despliegue

### PR 1: ciclo de vida duradero

- Esta nota de diseño.
- Esquema de SQLite compartido, generación de Kysely, almacén y depuración tras 30 días.
- Servicio de aprobación del Gateway, puente de espera del entorno de ejecución y gestión de elementos huérfanos tras reinicios.
- `approval.get/resolve` unificado.
- Adaptadores de métodos de ejecución/plugins.
- Pruebas de prevalencia de la primera respuesta, idempotencia, expiración, autorización y consumo.
- Aún no hay cambios en el comportamiento de la interfaz de usuario ni de los canales.

### PR 2: acciones tipadas y devoluciones de llamada de canales

- Acciones tipadas de aprobación, URL y aplicación web.
- Constructores de presentación del núcleo y exportaciones del SDK de plugins.
- Codificación de callbacks privada del transporte con tipo de propietario explícito.
- Referencias de callbacks duraderas y de tamaño fijo para identificadores canónicos que superen los límites del transporte.
- Migración de canales incluidos para abandonar la inferencia basada en texto de comandos e identificadores de aprobación.
- Verdad canónica de la primera respuesta en la superficie donde se hizo clic y actualizaciones terminales nativas activas con el mejor esfuerzo; la terminalización duradera de los mensajes de canal queda como trabajo posterior.
- Pruebas del SDK y de los canales incluidos.

### PR 3: enlace profundo de la interfaz de control

- Página de aprobación autenticada independiente y enrutamiento de inicio que tiene en cuenta la ruta base.
- Vinculación al Gateway servidor sin modificar la selección remota guardada del operador.
- Espacio de nombres HTTP de aprobaciones propiedad del núcleo, incluidos identificadores similares a recursos.
- Carga útil de URL generada por el Gateway y sondeo del estado pendiente hasta que se incorporen los eventos del ciclo de vida.
- Pruebas de ancho móvil, reconexión, respuestas simultáneas, recarga y ruta montada.

### PR 4: clientes nativos

- Las superficies de revisión de iOS y Android usan `approval.get/resolve` según el tipo; watchOS retransmite solicitudes seguras para el revisor y decisiones a través del iPhone emparejado.
- Watch ofrece las decisiones de ejecución admitidas por su contrato de retransmisión compacto: permitir una vez y denegar.
- La verdad terminal canónica de la primera respuesta sustituye al estado local de la decisión intentada.
- Las confirmaciones de resolución perdidas o ambiguas bloquean los controles hasta la relectura canónica.
- Las instancias anteriores publicadas de Gateway v4 conservan la revisión de ejecución mediante un mecanismo alternativo limitado basado en el método heredado; el estado terminal conservado entre superficies requiere los métodos unificados.
- Las advertencias para el revisor y el contexto del propietario permanecen visibles en iPhone, Watch y Android.
- Pruebas nativas unitarias, de compilación y de plataforma.

### PR 5: propagación del ciclo de vida a los ancestros

- Entrega pendiente/terminal de `session.approval` desde la instantánea de audiencia persistida en el PR 1.
- Suscripción a la sesión exacta, reproducción tras la reconexión y lápidas terminales sin mutar la transcripción ni activar al agente.
- Los callbacks del ciclo de vida se ejecutan después de la inserción/CAS duradera y nunca se convierten en autoridad de aprobación.
- Pruebas de subagentes anidados y reconexión.

### PR 6: comportamiento de cierre seguro

- Migrar `node-invoke-plugin-policy.ts` y el intermediario de plugins integrado para abandonar la autoridad duplicada.
- Semántica estricta de tiempo de espera, datos malformados, ausencia de ruta, vinculación y consumo de permitir una vez.
- Marcar como obsoleta la configuración permisiva de tiempo de espera publicada sin respetarla después de que una solicitud quede pendiente.
- Pruebas de contención entre múltiples superficies e inyección de fallos.

### Trabajo posterior: limpieza duradera de mensajes remotos

- Persistir los localizadores de entrega reenviada y terminalizar todos los mensajes de canal entregados después de un reinicio.
- Mantener este ciclo de vida del transporte separado de la autoridad de aprobación canónica y de las acciones de presentación tipadas.

## Pruebas

Cobertura específica requerida:

- La reapertura de SQLite conserva las proyecciones pendientes y terminales.
- Dos resolutores concurrentes producen exactamente un ganador de CAS.
- El reintento de la misma decisión se completa correctamente de forma idempotente; un reintento conflictivo devuelve el ganador registrado.
- La resolución en la fecha límite o después de ella no puede aprobar.
- `allow-once` se puede consumir exactamente una vez sin borrar el estado de auditoría terminal.
- El inicio cancela las épocas de ejecución anteriores.
- La consulta y la resolución no autorizadas no revelan la existencia del registro.
- Lista explícita de revisores permitidos y comportamiento general de `operator.approvals` emparejado.
- Los métodos heredados de ejecución y plugins comparten el mismo almacén.
- Esquemas de solicitud, listado, consulta y resolución del Gateway, y cargas útiles de eventos aditivas.
- Normalización de acciones tipadas, representación alternativa, exportaciones del SDK y cambios en los canales incluidos.
- La codificación de callbacks de Telegram contiene datos privados del transporte y ninguna inferencia basada en cadenas de comandos.
- Hijo directo, propietarios de controlador/solicitante ramificados, propietarios anidados, reasignación, mecanismo alternativo para el campo de sesión, ciclo y límite de tamaño de audiencia.
- Las matrices de audiencia solicitada y terminal son idénticas.
- Las proyecciones del propietario no provocan ninguna mutación de la transcripción ni activación del agente.
- La ruta de la interfaz de control funciona en `/` y en una ruta base configurada; la actualización muestra la verdad pendiente o terminal.
- Las respuestas simultáneas de la interfaz de control y Telegram muestran un ganador y «resuelto en otro lugar» en el perdedor.
- Los identificadores de aprobación nativos y los identificadores de propietario del Gateway conservan exactamente los bytes UTF-8 durante el enrutamiento y la reconciliación.
- La negociación de la familia RPC nativa fija una familia canónica o heredada por cada ruta de Gateway admitida y nunca cambia silenciosamente a una versión anterior después de usarla.
- Las confirmaciones de resolución nativas perdidas bloquean las acciones hasta la relectura canónica; una relectura fallida no puede inventar un ganador ni confirmar una actualización de Watch.
- La correlación de solicitudes de instantáneas de Watch solo se acepta para el propietario exacto del Gateway emparejado y una relectura canónica completada del iPhone.
- Prueba del recorrido del usuario mediante Testbox/Crabbox, incluida una página de aprobación con ancho móvil, la limpieza de acciones de Telegram y un recorrido completo de pendiente/resolución/perdedor tardío entre Android, iPhone y Watch.

## Observabilidad

Emitir registros estructurados de transiciones, sin contenido, con el identificador de aprobación, el tipo, la clave de sesión de origen, el estado, el motivo y la latencia. Nunca registrar la vista previa ni la vinculación sin procesar.

Realizar el seguimiento de:

- recuento de solicitudes por tipo;
- recuento de estados terminales por tipo/estado/motivo;
- indicador de pendientes;
- latencia desde la solicitud hasta el estado terminal;
- resultados de carreras de resolución: ganador, reintento idempotente, conflicto, expirado;
- recuento de rutas de entrega y denegaciones por ausencia de ruta;
- cancelaciones de solicitudes huérfanas al inicio;
- tamaño de la audiencia.

Una transición confirmada es correcta incluso si falla la entrega posterior del evento. Los suscriptores del ciclo de vida se recuperan mediante la reproducción del PR 5 y la consulta canónica. La terminalización duradera de los mensajes de canal sigue siendo el trabajo posterior independiente descrito anteriormente.

## Decisiones pendientes

1. **Origen de la interfaz de control accesible externamente.** Cada instantánea incluye la ruta relativa estable `urlPath`. Solo se puede anunciar una URL absoluta desde una ubicación almacenada en caché de Tailscale Serve/Funnel después de que la exposición del Gateway se complete correctamente; `allowedOrigins`, los encabezados Host de las solicitudes, `gateway.remote.url` y los candidatos de bucle invertido/LAN destinados únicamente a visualización no son orígenes canónicos. Telegram puede usar su contenedor de Mini App autenticado para conservar la ruta de aprobación durante el arranque. Los proxies inversos arbitrarios siguen limitados a rutas relativas hasta que exista un contrato explícito de URL pública revisado por separado. Nunca se debe permitir que un canal adivine el origen.
2. **Transición de compatibilidad al tiempo de espera estricto de ejecución.** Los tiempos de espera de aprobación de plugins ahora aplican el cierre seguro y `timeoutBehavior` está obsoleto. El contrato publicado restante `askFallback` requiere una revisión explícita del propietario y de seguridad, registro de cambios, documentación y una decisión de migración u obsolescencia antes de dejar de autorizar la ejecución cuando se agote el tiempo de espera de una solicitud pendiente.
3. **Modo integrado sin Gateway.** Recomendación: mantenerlo inicialmente solo en modo local y convertirlo después en cliente del servicio canónico cuando exista un Gateway. No anunciar un enlace profundo que ningún servidor pueda resolver.
