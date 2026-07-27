---
read_when:
    - Migración de los datos de ejecución, la caché, las transcripciones, el estado de las tareas o los archivos temporales de OpenClaw a SQLite
    - Diseño de migraciones de doctor desde archivos JSON o JSONL heredados
    - Cambiar el comportamiento de las copias de seguridad, la restauración, el VFS o el almacenamiento de los workers
    - Eliminar bloqueos de sesión, depuración, truncamiento o rutas de compatibilidad con JSON
summary: Plan de migración para convertir SQLite en la capa principal de estado duradero y caché, manteniendo la configuración respaldada por archivos
title: Refactorización del estado con prioridad en la base de datos
x-i18n:
    generated_at: "2026-07-26T04:50:26Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# Refactorización del estado con prioridad para la base de datos

## Decisión

Usar una disposición de SQLite de dos niveles:

- Base de datos global: `~/.openclaw/state/openclaw.sqlite`
- Base de datos del agente: una base de datos SQLite por agente para el espacio de trabajo,
  la transcripción, el VFS, los artefactos y el estado de ejecución grande propiedad del agente
- La configuración sigue respaldada por archivos: `openclaw.json` permanece fuera de la
  base de datos. Los perfiles de autenticación de ejecución se trasladan a SQLite; los archivos de
  credenciales de proveedores externos o de la CLI siguen gestionados por sus propietarios fuera de la base de datos de OpenClaw.

La base de datos global es la base de datos del plano de control. Es propietaria del descubrimiento de agentes,
el estado compartido del Gateway, el emparejamiento, el estado de dispositivos/nodos, los registros de tareas y flujos, el estado de los plugins,
el estado de ejecución del planificador, los metadatos de las copias de seguridad y el estado de las migraciones.

La base de datos del agente es la base de datos del plano de datos. Es propietaria de los metadatos
de sesión del agente, el flujo de eventos de transcripción, el espacio de trabajo del VFS o el espacio de nombres temporal,
los artefactos de herramientas, los artefactos de ejecución y los datos de caché locales del agente que pueden buscarse e indexarse.

Esto proporciona una vista global duradera sin forzar que los espacios de trabajo grandes de los agentes,
las transcripciones y los datos binarios temporales entren en la vía compartida de escritura del Gateway.

## Contrato estricto

Esta migración tiene una única forma de ejecución canónica:

- Las filas de sesión conservan únicamente los metadatos de sesión. No deben conservar
  `transcriptLocator`, rutas de archivos de transcripción, rutas JSONL relacionadas, rutas de bloqueo,
  metadatos de depuración ni punteros de compatibilidad de la era de los archivos.
- La identidad de la transcripción siempre es una identidad de SQLite: `{agentId, sessionId}` más
  metadatos opcionales del tema cuando el protocolo los requiera.
- `sqlite-transcript://...` no es una identidad de ejecución ni de protocolo. El código nuevo no debe
  derivar, conservar, pasar, analizar ni migrar localizadores de transcripciones. La ejecución y
  las pruebas no deben contener ningún seudolocalizador; la documentación puede mencionar la cadena
  únicamente para prohibirla.
- Los elementos heredados `sessions.json`, el JSONL de transcripciones, `.jsonl.lock`, la depuración, el truncamiento
  y la lógica antigua de rutas de sesión pertenecen únicamente a la ruta de migración/importación de doctor.
- Los alias heredados de configuración de sesiones pertenecen únicamente a la migración de doctor. La ejecución
  no interpreta `session.idleMinutes`, `session.resetByType.dm` ni
  los alias entre agentes de `agent:main:*` para la sesión principal de otro agente configurado.
- La identidad de enrutamiento de sesiones es un estado relacional tipado. Las rutas activas de ejecución y de la interfaz
  deben leer `sessions.session_scope`, `sessions.account_id`,
  `sessions.primary_conversation_id`, `conversations` y
  `session_conversations`; no deben analizar `session_key` ni extraer
  de `session_entries.entry_json` la identidad del proveedor, excepto como reflejo de compatibilidad
  mientras se eliminan los sitios de llamada antiguos.
- Los marcadores de mensajes directos a nivel de canal, como `dm` frente a `direct`, forman parte del vocabulario de enrutamiento,
  no son localizadores de transcripciones ni identificadores de compatibilidad con el almacenamiento en archivos.
- La configuración heredada de controladores de hooks pertenece únicamente a las superficies de advertencia/migración de doctor.
  La ejecución no debe cargar `hooks.internal.handlers`; los hooks se ejecutan únicamente mediante
  los directorios de hooks descubiertos y los metadatos de `HOOK.md`.
- El inicio de la ejecución, las rutas activas de respuesta, Compaction, el restablecimiento, la recuperación, los diagnósticos,
  TTS, los hooks de memoria, los subagentes, el enrutamiento de comandos de plugins, los límites del protocolo y
  los hooks deben pasar `{agentId, sessionId}` por la ejecución.
- Las pruebas deben crear y comprobar filas de transcripciones de SQLite mediante
  `{agentId, sessionId}`. Las pruebas que solo demuestren el reenvío de rutas JSONL,
  la conservación de localizadores proporcionados por el llamador o la compatibilidad con archivos de transcripción
  deben eliminarse, salvo que cubran la importación de doctor, la materialización de
  material de soporte/depuración ajeno a sesiones o la forma del protocolo.
- `runEmbeddedPiAgent(...)`, las ejecuciones preparadas de trabajadores y el intento
  integrado interno no deben aceptar localizadores de transcripciones. Abren el gestor de transcripciones de SQLite
  mediante `{agentId, sessionId}` y pasan ese gestor a la sesión de agente
  compatible con PI internalizada, de modo que los llamadores obsoletos no puedan hacer que el ejecutor escriba
  transcripciones JSON/JSONL.
- Los diagnósticos del ejecutor deben almacenar los registros de seguimiento de ejecución/caché/carga útil en SQLite.
  Los diagnósticos de ejecución no deben exponer controles para sustituir archivos JSONL ni helpers genéricos
  de exportación de transcripciones JSONL; las exportaciones orientadas al usuario pueden materializar artefactos explícitos
  a partir de filas de la base de datos sin devolver nombres de archivo a la ejecución.
- El registro del flujo sin procesar usa `OPENCLAW_RAW_STREAM=1` junto con filas de diagnóstico de SQLite.
  El antiguo contrato del registrador de archivos de pi-mono `PI_RAW_STREAM`, `PI_RAW_STREAM_PATH` y
  `raw-openai-completions.jsonl` no forma parte de la ejecución ni de las pruebas de OpenClaw.
- La indexación de memoria de QMD no debe exportar las transcripciones de SQLite a archivos Markdown.
  QMD indexa únicamente los archivos de memoria configurados; la búsqueda en transcripciones de sesiones sigue
  respaldada por SQLite.
- La subruta del SDK de QMD es exclusiva de QMD para el código nuevo. Los helpers de indexación
  de transcripciones de sesiones de SQLite residen en `memory-core-host-engine-session-transcripts`; cualquier
  reexportación de QMD existe únicamente por compatibilidad y el código de ejecución no debe usarla.
- Los índices de memoria integrados residen en la base de datos del agente propietario. La configuración de ejecución y
  los contratos de ejecución resueltos no deben exponer `memorySearch.store.path`; doctor
  elimina esa clave de configuración heredada y el código actual pasa internamente
  `databasePath` del agente.

El trabajo de implementación debe seguir eliminando código hasta que estas afirmaciones sean ciertas
sin excepciones fuera de los límites de doctor/importación/exportación/depuración.

## Estado objetivo y progreso

### Objetivo estricto

- Una base de datos SQLite global es propietaria del estado del plano de control:
  `state/openclaw.sqlite`.
- Una base de datos SQLite por agente es propietaria del estado del plano de datos:
  `agents/<agentId>/agent/openclaw-agent.sqlite`.
- La configuración sigue respaldada por archivos. `openclaw.json` no forma parte de esta
  refactorización de la base de datos.
- Los archivos heredados son únicamente entradas para la migración de doctor.
- La ejecución nunca escribe ni lee JSONL de sesiones o transcripciones como estado activo.

### Estados objetivo

- `not-started`: el código de ejecución de la era de los archivos aún escribe estado activo.
- `migrating`: el código de doctor/importación puede trasladar datos de archivos a SQLite.
- `dual-read`: un puente temporal lee tanto SQLite como archivos heredados. Este estado
  está prohibido para esta refactorización, salvo que se documente explícitamente como
  exclusivo de doctor.
- `sqlite-runtime`: la ejecución solo lee y escribe en SQLite.
- `clean`: se eliminan las API y las pruebas de ejecución heredadas, y la protección evita
  regresiones.
- `done`: la documentación, las pruebas, las copias de seguridad, la migración de doctor y las comprobaciones de cambios demuestran el
  estado limpio.

### Estado actual

- Sesiones: `clean` para la ejecución. Las filas de sesión residen en la base de datos por agente,
  las API de ejecución usan `{agentId, sessionId}` o `{agentId, sessionKey}`, y
  `sessions.json` es una entrada heredada exclusiva de doctor.
- Transcripciones: `clean` para la ejecución. Los eventos, las identidades y las instantáneas de transcripción,
  así como los eventos de ejecución de trayectorias, residen en la base de datos por agente. La ejecución ya no
  acepta localizadores de transcripciones ni rutas de transcripciones JSONL.
- Ejecutor PI integrado: `clean`. Las ejecuciones PI integradas, los trabajadores preparados, Compaction
  y los bucles de reintento usan el ámbito de sesión de SQLite y rechazan identificadores de transcripciones obsoletos.
- Cron: `clean` para la ejecución. La ejecución usa `cron_jobs` y `task_runs` propiedad de Cron;
  las pruebas de ejecución usan la nomenclatura `storeKey` de SQLite, y las rutas de Cron de la era de los archivos permanecen
  únicamente en las pruebas de migración heredada de doctor.
- Registro de tareas: `clean`. Las filas de ejecución de tareas y flujos de tareas residen en
  `state/openclaw.sqlite`; se han eliminado los importadores SQLite auxiliares que no se publicaron.
- Estado de los plugins: `clean`. Las filas de estado/blobs de los plugins residen en la base de datos global
  compartida; existen protecciones contra los antiguos helpers SQLite auxiliares del estado de plugins.
- Memoria: `sqlite-runtime` para la memoria integrada y la indexación de transcripciones de sesiones.
  Las tablas de índices de memoria residen en la base de datos por agente, el estado de memoria de los plugins usa
  filas compartidas de estado de plugins, y los archivos de memoria heredados son entradas para la migración de doctor
  o contenido del espacio de trabajo del usuario.
- Copia de seguridad: `sqlite-runtime`. La copia de seguridad prepara instantáneas compactas de SQLite, omite los
  archivos auxiliares WAL/SHM activos, verifica la integridad de SQLite y registra las ejecuciones de copias de seguridad en la
  base de datos global.
- Configuración del espacio de trabajo: `sqlite-runtime`. La finalización de la configuración, las certificaciones del espacio de trabajo
  y los hashes de arranque generados residen en tablas SQLite compartidas y tipadas. La ejecución
  no lee ni escribe el JSON retirado del espacio de trabajo ni los archivos auxiliares `.attested`;
  doctor se encarga de su importación validada y su eliminación verificada.
- Migración de doctor: `migrating`, de forma intencionada. Doctor importa los almacenes heredados JSON,
  JSONL y auxiliares retirados a SQLite, registra las ejecuciones/fuentes de migración
  y elimina las fuentes migradas correctamente.
- Aprobaciones de ejecución: `file-runtime`. TypeScript y macOS aún leen y escriben
  `exec-approvals.json` del directorio de estado activo; el esquema reservado
  `exec_approvals_config` todavía no tiene propietario en la ejecución. Una transición futura debe
  añadir la importación de doctor en el mismo estado y trasladar ambas ejecuciones a la vez.
- Scripts E2E: `clean` para la cobertura de ejecución. La inicialización de MCP en Docker escribe filas de SQLite.
  El script de Docker para el contexto de ejecución crea JSONL heredado únicamente dentro de la
  semilla de migración de doctor e identifica explícitamente la ruta heredada del índice de sesiones.

### Trabajo restante

- [x] Cambiar el nombre de las variables de almacén de las pruebas de ejecución de Cron para que dejen de usar `storePath`, salvo
      que sean entradas heredadas de doctor.
      Archivos: `src/cron/service.test-harness.ts`,
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`,
      `src/cron/service/timer.regression.test.ts`,
      `src/cron/service/ops.test.ts`, `src/cron/service/store.test.ts`,
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`,
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`,
      `src/cron/store.test.ts`.
      Prueba: `pnpm check:database-first-legacy-stores`; `rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`.
- [x] Eliminar o cambiar el nombre de los mocks obsoletos de pruebas de exportación de la era de los archivos.
      Archivo: `src/auto-reply/reply/commands-export-test-mocks.ts`.
      Prueba: `rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`.
- [x] Hacer que la semilla JSONL heredada del contexto de ejecución de Docker sea claramente exclusiva de doctor.
      Archivo: `scripts/e2e/session-runtime-context-docker-client.ts`.
      Prueba: `rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` muestra únicamente
      `seedBrokenLegacySessionForDoctorMigration`.
- [x] Mantener alineados los tipos generados de Kysely después de cualquier cambio en el esquema.
      Archivos: `src/state/openclaw-state-schema.sql`,
      `src/state/openclaw-agent-schema.sql`,
      `src/state/*generated*`.
      Prueba: no hubo cambios de esquema en esta pasada; `pnpm db:kysely:check`;
      `pnpm lint:kysely`.
- [x] Volver a ejecutar las pruebas específicas de los almacenes, comandos y scripts modificados.
      Prueba: `pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`; `git diff --check`.
- [x] Antes de declarar `done`, ejecutar la comprobación de cambios o una prueba amplia remota.
      Prueba: `pnpm check:changed --timed -- <changed extension paths>` se completó correctamente en
      la ejecución `run_3f1cabf6b25c` de Hetzner Crabbox tras la configuración temporal de Node 24/pnpm y
      el enrutamiento explícito de rutas para el espacio de trabajo sincronizado sin `.git`.

### No introducir regresiones

- Ningún localizador de transcripciones.
- Ningún archivo de sesión activo.
- Ningún fixture de prueba JSONL ficticio, excepto en las pruebas de migración heredada de doctor.
- Ningún acceso directo a SQLite donde se espere Kysely.
- Ninguna nueva migración de base de datos de la era de los archivos. El esquema global permanece en la versión `1`.
  El esquema publicado por agente de la versión `1` tiene una única migración de ejecución acotada a la
  versión `2` para identidades estables de fuentes de memoria.

## Supuestos de lectura del código

No hay decisiones de producto pendientes que bloqueen este plan. La implementación debe
continuar con estos supuestos:

- Usar `node:sqlite` directamente y requerir un entorno de ejecución Node seguro ante el restablecimiento de WAL
  (22.22.3+, 24.15+ o 25.9+) para esta ruta de almacenamiento.
- Mantener exactamente un archivo de configuración normal. No trasladar la configuración, los manifiestos de plugins
  ni los espacios de trabajo de Git a SQLite en esta refactorización.
- No se requieren archivos de compatibilidad en tiempo de ejecución. Los archivos JSON y JSONL heredados son
  únicamente entradas de migración. Los archivos auxiliares de SQLite locales de la rama nunca se publicaron y se
  eliminan en lugar de importarse.
- `openclaw doctor --fix` se encarga de la migración de archivos heredados a la base de datos. El inicio del entorno de ejecución
  solo se encarga de actualizaciones acotadas entre versiones publicadas del esquema de SQLite;
  no debe importar el estado de la época de los archivos.
- La compatibilidad de credenciales sigue la misma regla: las credenciales en tiempo de ejecución residen en
  SQLite. Los archivos antiguos `auth-profiles.json`, los `auth.json` por agente y los
  `credentials/oauth.json` compartidos son entradas de migración de doctor y se eliminan
  después de importarlos.
- El estado generado del catálogo de modelos se respalda en la base de datos. El código en tiempo de ejecución no debe escribir
  `agents/<agentId>/agent/models.json`; los archivos `models.json` existentes son entradas heredadas
  de doctor y se eliminan después de importarlos en `agent_model_catalogs`.
- El entorno de ejecución no debe migrar, normalizar ni crear puentes para los localizadores de transcripciones. La identidad
  activa de la transcripción es `{agentId, sessionId}` en SQLite. Las rutas de archivos son
  únicamente entradas heredadas de doctor, y `sqlite-transcript://...` debe desaparecer de
  las superficies del entorno de ejecución, el protocolo, los hooks y los plugins, en lugar de tratarse como un
  identificador de frontera.
- Las lecturas de transcripciones desde SQLite en tiempo de ejecución no ejecutan migraciones antiguas de la estructura de entradas JSONL ni
  reescriben transcripciones completas por compatibilidad. La normalización de entradas heredadas permanece en
  utilidades explícitas de doctor/importación. Doctor normaliza los archivos de transcripción JSONL
  heredados antes de insertar las filas de SQLite; las filas actuales del entorno de ejecución
  ya se escriben con el esquema de transcripción actual. La exportación de trayectorias/sesiones
  lee esas filas tal como están y no debe realizar migraciones heredadas durante la exportación.
- Los auxiliares heredados de análisis/migración de transcripciones JSONL son exclusivos de doctor. El código de formato
  de transcripciones en tiempo de ejecución solo construye el contexto actual de transcripciones de SQLite; doctor
  se encarga de actualizar las entradas JSONL antiguas antes de insertar las filas.
- Se eliminó el antiguo auxiliar de transmisión de transcripciones JSONL gestionado por el entorno de ejecución. El código de
  importación de doctor se encarga de las lecturas explícitas de archivos heredados; el historial de sesiones en tiempo de ejecución
  lee las filas de SQLite.
- Los enlaces del servidor de aplicaciones de Codex usan el `sessionId` de OpenClaw como clave canónica
  en el espacio de nombres del estado del plugin de Codex. `sessionKey` son metadatos para
  el enrutamiento/la visualización y no deben sustituir el identificador duradero de la sesión ni resucitar
  la identidad del archivo de transcripción.
- Los motores de contexto reciben directamente el contrato actual del entorno de ejecución. El registro
  no debe envolver los motores con adaptadores de reintento que eliminen `sessionKey`,
  `transcriptScope` o `prompt`; los motores que no puedan aceptar los parámetros actuales
  centrados en la base de datos deben fallar de forma explícita en lugar de conectarse mediante un puente.
- La salida de la copia de seguridad debe seguir siendo un único archivo comprimido. El contenido de la base de datos debe entrar
  en ese archivo como instantáneas compactas de SQLite, no como archivos auxiliares WAL activos sin procesar.
- La búsqueda de transcripciones es útil, pero no es necesaria para la primera versión centrada en la base de datos.
  Diseñar el esquema de modo que FTS pueda añadirse más adelante.
- La ejecución de workers debe seguir siendo experimental y permanecer detrás de ajustes mientras se estabiliza el límite
  de la base de datos.

## Hallazgos de la lectura del código

La rama actual ya ha superado la etapa de prueba de concepto. La base de datos
compartida existe, Node `node:sqlite` está conectado mediante un pequeño auxiliar del entorno de ejecución y
los antiguos almacenes ahora escriben en `state/openclaw.sqlite` o en la base de datos
`openclaw-agent.sqlite` propietaria.

El trabajo restante no consiste en elegir SQLite, sino en mantener limpio el nuevo límite
y eliminar cualquier interfaz con forma de compatibilidad que todavía se parezca al antiguo
mundo de los archivos:

- La sesión `storePath` ya no es una identidad del entorno de ejecución, una estructura de fixture de prueba ni
  un campo de carga útil de estado. Las pruebas del entorno de ejecución y del puente ya no contienen el
  nombre de contrato `storePath`; el código de doctor/migración se encarga de ese vocabulario heredado.
- Las escrituras de sesión ya no pasan por la antigua cola `store-writer.ts`
  dentro del proceso. Las escrituras de parches de SQLite se preparan fuera de la transacción y luego usan una transacción breve,
  síncrona, de validación/aplicación, con detección explícita de conflictos.
- El descubrimiento de rutas heredadas aún tiene usos válidos para la migración, pero el código en tiempo de ejecución debe
  dejar de tratar `sessions.json` y los archivos JSONL de transcripciones como posibles destinos de
  escritura.
- Las tablas propiedad de los agentes residen en bases de datos SQLite por agente. La base de datos global conserva
  las filas del registro/plano de control; la identidad de la transcripción es `{agentId, sessionId}` en
  las filas de transcripciones por agente. El código en tiempo de ejecución no debe persistir las rutas de los archivos de transcripción
  ni migrar los localizadores de transcripciones.
- Doctor ya importa varios archivos heredados. La limpieza consiste en convertir esto en una
  única implementación explícita de migración invocada por doctor, con un informe
  de migración persistente.

No hay preguntas adicionales sobre el producto que bloqueen la implementación.

## Estructura actual del código

La rama ya cuenta con una base SQLite compartida real:

- La versión mínima del entorno de ejecución ahora requiere una compilación de Node segura para el restablecimiento de WAL: 22.22.3+,
  24.15+ o 25.9+. `package.json`, la protección del entorno de ejecución de la CLI, los valores predeterminados del instalador,
  el localizador del entorno de ejecución de macOS, la CI y la documentación pública de instalación coinciden.
- `src/state/openclaw-state-db.ts` abre `openclaw.sqlite`, configura WAL,
  `synchronous=NORMAL`, `busy_timeout=30000`, `foreign_keys=ON` y aplica
  el módulo de esquema generado derivado de
  `src/state/openclaw-state-schema.sql`.
- Los tipos de tabla de Kysely y los módulos de esquema del entorno de ejecución se generan a partir de bases de datos
  SQLite desechables creadas desde los archivos `.sql` confirmados; el código del entorno de ejecución ya no
  mantiene cadenas de esquema copiadas y pegadas para bases de datos globales, por agente o de
  captura de proxy.
- Los almacenes del entorno de ejecución derivan los tipos de fila seleccionados e insertados de esas interfaces
  `DB` generadas de Kysely, en lugar de reproducir manualmente las formas de las filas de SQLite. El SQL sin procesar
  sigue limitado a la aplicación de esquemas, pragmas y DDL exclusivo de migraciones.
- El esquema global de SQLite permanece en `user_version = 1`. El esquema por agente
  está en la versión `2`; su función de apertura migra atómicamente la clave de origen de memoria de la versión distribuida `1`
  a una identidad entera estable. La importación de archivos a la base de datos
  permanece en el código de doctor.
- La propiedad relacional se aplica donde el límite de propiedad es canónico:
  las filas de migración de orígenes se eliminan en cascada desde `migration_runs`, el estado de entrega de tareas
  se elimina en cascada desde `task_runs` y las filas de identidad de transcripciones se eliminan en cascada desde
  los eventos de transcripción.
- Las tablas compartidas actuales incluyen `agent_databases`,
  `auth_profile_stores`, `auth_profile_state`,
  `plugin_state_entries`, `plugin_blob_entries`, `media_blobs`,
  `skill_uploads`, `capture_sessions`, `capture_events`, `capture_blobs`,
  `sandbox_registry_entries`, `cron_jobs`, `commitments`,
  `delivery_queue_entries`, `model_capability_cache`,
  `workspace_setup_state`, `workspace_path_aliases`, `workspace_attestations`,
  `workspace_generated_bootstrap_hashes`, `native_hook_relay_bridges`,
  `current_conversation_bindings`, `plugin_binding_approvals`,
  `tui_last_sessions`, `acp_sessions`, `acp_replay_sessions`,
  `acp_replay_events`, `task_runs`, `task_delivery_state`, `flow_runs`,
  `subagent_runs`, `migration_runs` y `backup_runs`.
- El estado arbitrario propiedad de los plugins no obtiene tablas tipadas propiedad del host. Los
  plugins instalados usan `plugin_state_entries` para cargas útiles JSON con versiones y
  `plugin_blob_entries` para bytes, con propiedad de espacio de nombres/clave, limpieza por TTL,
  copia de seguridad y registros de migración de plugins. El estado de orquestación de plugins propiedad del host
  aún puede tener tablas tipadas cuando el host posee el contrato de consulta, como
  `plugin_binding_approvals`.
- Las migraciones de plugins son migraciones de datos sobre espacios de nombres propiedad de los plugins, no migraciones
  del esquema del host. Un plugin puede migrar sus propias entradas de estado/blob con versiones
  mediante un proveedor de migraciones, y el host registra el estado del origen y de la ejecución en el
  registro normal de migraciones. Las nuevas instalaciones de plugins no requieren cambiar
  `openclaw-state-schema.sql`, salvo que el propio host asuma la propiedad de un
  nuevo contrato entre plugins.
- `src/state/openclaw-agent-db.ts` abre
  `agents/<agentId>/agent/openclaw-agent.sqlite`, registra la base de datos en la
  base de datos global y posee las tablas locales del agente de sesión, transcripción, VFS, artefactos, caché
  e índice de memoria. El descubrimiento compartido del entorno de ejecución ahora lee el registro
  `agent_databases` tipado generado, en lugar de volver a implementar esa consulta en cada
  lugar de llamada.
- Las bases de datos globales y por agente registran una fila `schema_meta` con el rol de la base de datos,
  la versión del esquema, las marcas de tiempo y el id. del agente para las bases de datos de agentes. La base de datos global
  permanece en `user_version = 1`; las bases de datos por agente usan la versión `2` después de la migración acotada
  de identidad del origen de memoria.
- La identidad de sesión por agente ahora tiene una tabla raíz canónica `sessions` con clave
  `session_id`, con `session_key`, `session_scope`, `account_id`,
  `primary_conversation_id`, marcas de tiempo, campos de visualización, metadatos del modelo,
  id. del arnés y vínculos de elemento superior/generación como columnas consultables. `session_routes`
  es el índice único de ruta activa desde `session_key` hasta la
  `session_id` actual, de modo que una clave de ruta pueda trasladarse a una sesión duradera nueva sin
  hacer que las lecturas en caliente tengan que elegir entre filas `sessions.session_key` duplicadas. La antigua
  carga útil con forma de compatibilidad `session_entries.entry_json` depende de la
  raíz duradera `session_id` mediante una clave externa; ya no es la única
  representación de una sesión en el nivel del esquema.
- La identidad de conversaciones externas por agente también es relacional:
  `conversations` almacena la identidad normalizada de proveedor/cuenta/conversación y
  `session_conversations` vincula una sesión de OpenClaw con una o más conversaciones
  externas. Esto abarca las sesiones de MD principal compartida en las que varios pares pueden
  asignarse intencionadamente a una sesión sin falsear `session_key`. SQLite también
  impone la unicidad de la identidad natural del proveedor, de modo que la misma tupla
  canal/cuenta/tipo/par/hilo no pueda bifurcarse entre distintos id. de conversación.
  Los pares directos del entorno principal compartido se vinculan con un rol `participant`, de modo que una
  sesión de OpenClaw pueda representar varios pares de MD externos sin relegar
  los pares anteriores a filas relacionadas imprecisas. `sessions.primary_conversation_id` todavía
  apunta al destino de entrega tipado actual. Las columnas cerradas de enrutamiento/estado
  se aplican mediante restricciones `CHECK` de SQLite, en lugar de depender únicamente de
  uniones de TypeScript.
  La proyección de sesiones del entorno de ejecución elimina las réplicas de enrutamiento de compatibilidad de
  `session_entries.entry_json` antes de aplicar las columnas tipadas de sesión/conversación,
  de modo que las cargas útiles JSON obsoletas no puedan reactivar destinos de entrega.
  Del mismo modo, el enrutamiento de anuncios de subagentes requiere el contexto de entrega tipado de SQLite;
  ya no recurre a los campos de ruta de compatibilidad `SessionEntry`.
  La herencia explícita de entrega `chat.send` del Gateway lee el contexto de entrega tipado de SQLite
  en lugar de los campos de compatibilidad `origin`/`last*`.
  `tools.effective` también deriva el contexto de proveedor/cuenta/hilo de las filas tipadas
  de entrega/enrutamiento de SQLite, no de réplicas obsoletas de entradas de sesión `last*`.
  El contexto del prompt de eventos del sistema reconstruye los campos de canal/destino/cuenta/hilo a partir de
  campos de entrega tipados en lugar de réplicas `origin`.
  El asistente compartido `deliveryContextFromSession` y el asignador de sesión a conversación
  ahora ignoran `SessionEntry.origin` por completo; solo los campos de entrega tipados
  y las filas de conversaciones relacionales pueden crear una identidad de ruta en caliente.
  La normalización de entradas de sesión del entorno de ejecución elimina `origin` antes de persistir o
  proyectar `entry_json`, y los metadatos entrantes escriben campos tipados de canal/chat
  y filas de conversaciones relacionales, en lugar de crear nuevas
  réplicas de origen.
- Los eventos de transcripción, las instantáneas de transcripción y los eventos del entorno de ejecución de trayectorias ahora
  hacen referencia a la raíz canónica por agente `sessions` y se eliminan en cascada al eliminar la sesión.
  Las filas de identidad/idempotencia de transcripción siguen eliminándose en cascada desde la
  fila exacta del evento de transcripción.
- Los índices del núcleo de memoria ahora usan tablas explícitas de la base de datos del agente
  `memory_index_meta`, `memory_index_sources`, `memory_index_chunks` y
  `memory_embedding_cache`, con `memory_index_state` para hacer seguimiento de los cambios de revisión.
  Los índices secundarios opcionales de FTS/vectores se denominan `memory_index_chunks_fts` y
  `memory_index_chunks_vec`, en lugar de las tablas genéricas `meta`, `files`, `chunks`,
  `chunks_fts` o `chunks_vec`. Los nombres canónicos conservan la forma actual
  de las filas de ruta/origen y la compatibilidad de las incrustaciones serializadas. Estas tablas
  son cachés derivadas/de búsqueda, no almacenamiento canónico de transcripciones; pueden
  eliminarse y reconstruirse a partir de los archivos del espacio de trabajo de memoria y los orígenes configurados.
  Al abrir un índice de memoria distribuido con nombres genéricos, se migran sus metadatos, orígenes,
  fragmentos y caché de incrustaciones a las tablas canónicas; las tablas derivadas de FTS/vectores
  se reconstruyen con sus nombres canónicos.
- El estado de recuperación de ejecuciones de subagentes ahora reside en filas compartidas tipadas `subagent_runs`
  con claves indexadas de sesión secundaria, solicitante y controladora. El antiguo archivo
  `subagents/runs.json` solo sirve como entrada de limpieza para Doctor. Sus entradas de ejecución son
  estado de recuperación transitorio, por lo que Doctor registra el recibo de retirada y
  descarta el archivo sin importarlo. Como un archivo no puede demostrar si
  sus entradas están activas u obsoletas después de depurar las filas de SQLite, los operadores
  deben dejar que las ejecuciones activas de la era de los archivos finalicen antes de actualizar más allá de este límite.
- Los enlaces de conversaciones actuales ahora residen en filas compartidas tipadas
  `current_conversation_bindings` con clave de id. de conversación normalizado, con
  columnas de agente/sesión de destino, tipo de conversación, estado, caducidad y metadatos
  almacenados como columnas relacionales en lugar de un registro de enlace opaco duplicado.
  La clave de enlace duradera incluye el tipo de conversación normalizado para que
  las referencias directas/de grupo/de canal no puedan colisionar, y SQLite rechaza los valores no válidos de tipo/estado
  del enlace. El antiguo archivo
  `bindings/current-conversations.json` solo sirve como entrada de migración para doctor.
- La recuperación de la cola de entrega ahora superpone columnas tipadas de la cola para canal, destino,
  cuenta, sesión, reintento, error, envío de plataforma y estado de recuperación sobre el
  JSON de reproducción. `entry_json` conserva las cargas útiles de reproducción, los hooks y la carga útil
  de formato, pero las columnas tipadas son la fuente autoritativa para el enrutamiento/estado en caliente de la cola.
- Los punteros de restauración de la última sesión de la TUI ahora residen en filas compartidas tipadas
  `tui_last_sessions` con clave del ámbito con hash de conexión/sesión de la TUI.
  El entorno de ejecución solo lee y escribe SQLite, actualiza o inserta atómicamente cada ámbito y
  excluye las sesiones de Heartbeat. `openclaw doctor --fix` valida estrictamente el
  antiguo archivo JSON de la TUI, conserva las filas de SQLite más recientes, verifica el resultado canónico
  y elimina el archivo heredado sin cambios en lugar de dejar un archivo de respaldo.
- Los hashes de despliegue de comandos de Discord ahora residen en el almacén SQLite compartido
  de estado de plugins. El entorno de ejecución solo lee y escribe claves exactas con ámbito de aplicación. Doctor
  elimina el archivo heredado reconstruible `discord/command-deploy-cache.json`
  sin importarlo, de modo que el siguiente inicio realiza una única reconciliación canónica.
- Las preferencias predeterminadas de TTS ahora residen en filas SQLite compartidas de estado de plugins con claves bajo el
  plugin `speech-core`. El antiguo archivo `settings/tts.json` solo sirve como entrada de migración
  para doctor; el entorno de ejecución ya no lee ni escribe archivos JSON de preferencias de TTS, y el
  solucionador de rutas heredadas reside en el módulo de migración de doctor.
- Los metadatos de destino de secretos ahora hacen referencia a almacenes, en lugar de fingir que cada
  destino de credenciales es un archivo de configuración. `openclaw.json` sigue siendo el almacén de configuración;
  los destinos de perfiles de autenticación usan filas SQLite tipadas `auth_profile_stores`, con
  credenciales estructuradas según el proveedor conservadas como cargas útiles JSON.
- La auditoría de secretos ya no examina los archivos retirados por agente `auth.json`. Doctor se encarga
  de advertir sobre ese archivo heredado, importarlo y eliminarlo.
- Los asistentes de rutas de perfiles de autenticación heredados ahora residen en el código heredado de doctor. Los asistentes de rutas
  de perfiles de autenticación del núcleo exponen la identidad del almacén de autenticación de SQLite y las ubicaciones de visualización,
  no las rutas del entorno de ejecución `auth-profiles.json` o `auth-state.json`.
- Los módulos del entorno de ejecución de recuperación de ejecuciones de subagentes y de caché de capacidades de modelos de OpenRouter
  ahora mantienen separados los lectores/escritores de instantáneas de SQLite de los asistentes de importación de JSON heredado
  exclusivos de doctor. Las capacidades de OpenRouter usan las filas genéricas tipadas
  `model_capability_cache` bajo `provider_id = "openrouter"`, en lugar de
  un único blob de caché opaco o una tabla del host específica del proveedor. El
  `taskName` de la ejecución del subagente se almacena en la columna tipada `subagent_runs.task_name`; la
  copia `payload_json` contiene datos de reproducción/depuración, no es el origen de los campos de visualización o
  búsqueda en caliente.
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` implementa un VFS de SQLite
  sobre la tabla `vfs_entries` de la base de datos del agente. Las lecturas de directorios, las exportaciones recursivas,
  las eliminaciones y los cambios de nombre usan intervalos de prefijos indexados `(namespace, path)`,
  en lugar de examinar un espacio de nombres completo o depender de la coincidencia de rutas `LIKE`.
- `src/agents/runtime-worker.entry.ts` crea almacenes SQLite por ejecución para VFS, artefactos de herramientas,
  artefactos de ejecución y caché con ámbito para los workers.
- La finalización de la inicialización del espacio de trabajo, la antigüedad de la atestación y los hashes
  de inicialización generados ahora residen en filas compartidas con tipos `workspace_setup_state`,
  `workspace_path_aliases`, `workspace_attestations` y
  `workspace_generated_bootstrap_hashes`, indexadas por la identidad canónica del espacio de trabajo.
  Los alias léxicos y de rutas reales persistentes mantienen estable la protección
  de espacios de trabajo desaparecidos después de que desaparezca un enlace simbólico
  configurado; los alias redirigidos aplican un cierre seguro. El entorno de ejecución ya no lee ni escribe
  `openclaw-workspace-state.json`, `.openclaw/workspace-state.json`, `workspace-attestations/*.attested`
  del directorio de estado ni archivos auxiliares `<workspace>.attested`
  del mismo nivel. `openclaw doctor --fix` valida y reclama las fuentes heredadas,
  las importa en SQLite con recibos de migración, verifica las filas canónicas
  y solo entonces elimina los archivos reclamados.
- El esquema compartido reserva una fila singleton `exec_approvals_config`, pero la
  transición del entorno de ejecución sigue pendiente. TypeScript y la aplicación complementaria de macOS aún usan
  el archivo JSON con ámbito de estado y deben migrar juntos a SQLite.
- La identidad del dispositivo de TypeScript ahora usa filas con tipos `device_identities`, y
  la importación del JSON heredado exclusiva de doctor se mantiene fuera del propietario del entorno de ejecución. La autenticación del dispositivo
  sigue respaldada por archivos a la espera de una migración coordinada del esquema y entre entornos de ejecución;
  `device_auth_tokens` permanece reservado para ese seguimiento.
- La caché de intercambio de tokens de GitHub Copilot usa la tabla compartida de estado de Plugin de SQLite
  en `github-copilot/token-cache/default`. Es un estado de caché propiedad del proveedor,
  por lo que intencionadamente no añade una tabla al esquema del host.
- La Compaction de GitHub Copilot ya no escribe archivos auxiliares
  `openclaw-compaction-*.json` del espacio de trabajo. El arnés llama al RPC de Compaction del historial del SDK para la
  sesión del SDK supervisada, y OpenClaw mantiene el estado duradero de las sesiones y transcripciones en
  SQLite en lugar de archivos marcadores de compatibilidad.
- El entorno de ejecución compartido de Swift (`OpenClawKit`) usa la misma
  forma `state/openclaw.sqlite#table/device_identities` y las mismas claves de fila para la identidad del
  dispositivo. Los archivos heredados de contenedores de Apple son importados por el propietario de la migración de Swift
  porque el Doctor de TypeScript no puede acceder a esos contenedores. La autenticación de dispositivos de Swift
  sigue respaldada por archivos para el seguimiento coordinado de autenticación.
- La identidad del dispositivo de Android y la autenticación del dispositivo almacenada en caché permanecen en almacenes locales de la aplicación. Requieren
  una migración separada propiedad de Android; las afirmaciones sobre el SQLite del host no
  describen el comportamiento actual de Android.
- El historial de paquetes recientes de notificaciones de Android usa filas con tipos
  `android_notification_recent_packages`. El entorno de ejecución ya no migra ni
  lee las antiguas claves CSV de SharedPreferences.
- La creación de la identidad del dispositivo aplica un cierre seguro cuando existe el archivo heredado `identity/device.json`,
  cuando la fila de identidad de SQLite no es válida o cuando no se puede abrir el almacén de identidades
  de SQLite. Doctor importa y elimina primero ese archivo, por lo que el inicio del entorno de ejecución
  no puede rotar silenciosamente la identidad de emparejamiento antes de la migración.
- La selección de la identidad del dispositivo es una clave de fila de SQLite, no un localizador de archivos JSON. Las pruebas
  y los auxiliares del Gateway pasan claves de identidad explícitas; solo la migración de doctor y la
  barrera de inicio con cierre seguro conocen el nombre de archivo retirado `identity/device.json`.
- La compatibilidad del restablecimiento de sesiones ahora reside en la migración de configuración de doctor:
  `session.idleMinutes` se traslada a `session.reset.idleMinutes`,
  `session.resetByType.dm` se traslada a `session.resetByType.direct`, y la
  política de restablecimiento del entorno de ejecución solo lee las claves de restablecimiento canónicas.
- La compatibilidad de la configuración heredada ahora reside en `src/commands/doctor/`. La validación normal
  de `readConfigFileSnapshot()` no importa detectores heredados de doctor
  ni anota problemas heredados; `runDoctorConfigPreflight()` añade esos problemas para
  que doctor los repare o notifique. El flujo de configuración de doctor importa
  `src/commands/doctor/legacy-config.ts`, y la reparación de identificadores antiguos de perfiles OAuth reside
  en
  `src/commands/doctor/legacy/oauth-profile-ids.ts`.
- Los comandos distintos de doctor no ejecutan automáticamente la reparación de configuración heredada. Por ejemplo,
  `openclaw update --channel` ahora falla ante una configuración heredada no válida y solicita al
  usuario que ejecute doctor, en lugar de importar silenciosamente el código de migración de doctor.
- Web push, APNs, Voice Wake, las comprobaciones de actualizaciones y el estado de la configuración ahora usan tablas compartidas de SQLite con tipos
  para suscripciones, claves VAPID, registros de Node, filas de activadores,
  filas de enrutamiento, estado de notificaciones de actualización y entradas del estado de configuración, en lugar de
  blobs JSON opacos completos. Las escrituras de Web Push y APNs realizan upsert únicamente en la fila
  de clave primaria afectada; el estado de configuración se concilia por ruta de configuración. Sus módulos del entorno de ejecución
  permanecen separados de los auxiliares de importación de JSON heredado exclusivos de Doctor.
- El entorno de ejecución de APNs solo lee y escribe `apns_registrations`. La acción explícita
  `openclaw doctor --fix` importa de forma estricta el elemento retirado
  `push/apns-registrations.json`, conserva las filas canónicas existentes, verifica
  la transacción, registra un recibo y elimina el JSON que contiene secretos.
  Los reintentos respaldados por recibos solo realizan la limpieza, mientras que
  `apns_registration_tombstones` cubren las invalidaciones anteriores a la primera reparación, de modo que
  las concesiones obsoletas del relé o los tokens de dispositivo no puedan reaparecer.
- La configuración del host de Node ahora usa una fila singleton con tipos en la base de datos SQLite compartida.
  El entorno de ejecución aplica un cierre seguro mientras permanezca el antiguo archivo `node.json` o una reclamación
  interrumpida; la acción explícita `openclaw doctor --fix` lo importa de forma estricta y lo elimina
  antes del uso normal del entorno de ejecución.
- El emparejamiento de dispositivos/nodos, el emparejamiento de canales, las listas de permitidos de canales y el estado de inicialización
  ahora usan filas SQLite con tipos en lugar de blobs JSON opacos completos. Las aprobaciones de vinculaciones
  de Plugin y el estado de trabajos Cron siguen la misma división: los módulos del entorno de ejecución exponen
  operaciones respaldadas por SQLite y auxiliares de instantáneas neutrales, y las escrituras de instantáneas de
  emparejamiento/inicialización y de aprobaciones de vinculaciones de Plugin concilian las filas por clave primaria
  en lugar de truncar tablas, mientras doctor importa y elimina los antiguos archivos JSON mediante
  módulos `src/commands/doctor/legacy/*`.
- Los registros de Plugins instalados ahora residen en el índice SQLite de Plugins instalados.
  La lectura y escritura de la configuración en el entorno de ejecución ya no migra ni conserva los antiguos
  datos de configuración creados `plugins.installs`; doctor importa esa forma de configuración heredada
  en SQLite antes del uso normal del entorno de ejecución.
- Las instantáneas de recuperación de credenciales de QQBot ahora residen en el estado de Plugin de SQLite en
  `qqbot/credential-backups`. El entorno de ejecución ya no escribe
  `qqbot/data/credential-backup*.json`; el contrato de doctor de QQBot importa y
  archiva esos archivos de copia de seguridad heredados desde el directorio de estado activo.
- La planificación de la recarga del Gateway compara instantáneas del índice SQLite de Plugins instalados en
  un espacio de nombres de diferencias interno `installedPluginIndex.installRecords.*`. Las decisiones de
  recarga del entorno de ejecución ya no envuelven esas filas en objetos de configuración `plugins.installs`
  ficticios.
- Las credenciales de las cuentas de Matrix ahora residen en el estado de Plugin de SQLite. El entorno de ejecución solo lee
  ese almacén canónico; Doctor importa, verifica y archiva los archivos retirados
  `credentials/matrix/credentials*.json` cuando se puede resolver su cuenta.
- Los módulos principales de emparejamiento y del entorno de ejecución de Cron ya no usan constructores de rutas JSON heredadas.
  El auxiliar obsoleto del SDK para rutas de emparejamiento permanece únicamente para compatibilidad de migración;
  la migración de estado de doctor es responsable de leer e importar sus archivos. Los módulos heredados propiedad de Doctor
  construyen las rutas de origen `pending.json`, `paired.json`, `bootstrap.json` y
  `cron/jobs.json` solo para pruebas de importación y migración. La normalización heredada
  de la forma de los trabajos Cron y la importación del historial JSONL residen en
  `src/commands/doctor/cron/`; la finalización del historial SQLite heredado se ejecuta al
  abrir la base de datos de estado.
- `src/commands/doctor/legacy/runtime-state.ts` importa archivos de estado JSON
  heredados, incluida la configuración del host de Node, en SQLite desde doctor. Los nuevos importadores de archivos
  heredados permanecen en `src/commands/doctor/legacy/`.
- `src/commands/doctor/state-migrations.ts` importa las transcripciones heredadas `sessions.json` y
  `*.jsonl` directamente en SQLite y elimina las fuentes importadas correctamente. Ya no
  prepara las transcripciones heredadas de la raíz mediante
  `agents/<agentId>/sessions/*.jsonl` ni crea un destino JSONL canónico antes de la
  importación.
- Las comprobaciones de integridad del estado de doctor ya no examinan directorios de sesiones heredados ni
  ofrecen eliminar archivos JSONL huérfanos. Los archivos de transcripciones heredados son únicamente entradas de migración,
  y el paso de migración es responsable de la importación y de la eliminación de las fuentes.
- La importación del registro heredado del entorno aislado reside en
  `src/commands/doctor/legacy/sandbox-registry.ts`; las lecturas y escrituras del registro
  activo del entorno aislado siguen realizándose exclusivamente en SQLite.
- La reparación del estado y la importación de transcripciones de sesiones heredadas reside en
  `src/commands/doctor/legacy/session-transcript-health.ts`; los módulos de comandos del entorno de ejecución
  ya no incluyen análisis de transcripciones JSONL ni código de reparación de la rama activa.

Aspectos destacados de la consolidación/eliminación completada:

- El estado del Plugin ahora usa la base de datos compartida `state/openclaw.sqlite`. Se eliminó el antiguo
  importador de archivo auxiliar `plugin-state/state.sqlite` local de la rama porque
  ese diseño de SQLite nunca se publicó. Los ayudantes de sondeo/prueba informan del
  `databasePath` compartido en lugar de exponer una ruta de SQLite específica del estado del Plugin.
- Las tablas de tiempo de ejecución de tareas y TaskFlow ahora se encuentran en la base de datos compartida
  `state/openclaw.sqlite` en lugar de `tasks/runs.sqlite` y
  `tasks/flows/registry.sqlite`; los antiguos importadores de archivos auxiliares se eliminaron por el
  mismo motivo de que el diseño no se publicó.
- `src/config/sessions/store.ts` ya no necesita `storePath` para los metadatos
  de entrada, las actualizaciones de rutas ni las lecturas de la fecha de actualización. La persistencia de comandos, la limpieza
  de sesiones de la CLI, la profundidad de los subagentes, las anulaciones de autenticación y la identidad de sesión
  de la transcripción usan las API de filas de agente/sesión. Las escrituras se aplican como parches de filas de SQLite
  con reintento optimista en caso de conflicto.
- La resolución del destino de sesión ahora expone destinos de base de datos por agente, no rutas
  `sessions.json` heredadas. El Gateway compartido, los metadatos de ACP, la reparación de rutas del doctor y
  `openclaw sessions` enumeran `agent_databases` además de los agentes configurados.
- El enrutamiento de sesiones del Gateway ahora usa `resolveGatewaySessionDatabaseTarget`; el
  destino devuelto contiene `databasePath` y claves candidatas de filas de SQLite en lugar
  de una ruta heredada al archivo del almacén de sesiones.
- Los tipos de tiempo de ejecución de sesión de canal ahora exponen `{agentId, sessionKey}` para
  las lecturas de la fecha de actualización, los metadatos de entrada y las actualizaciones de la última ruta. El antiguo
  tipo de compatibilidad `saveSessionStore(storePath, store)` se eliminó.
- Las superficies de sesión del tiempo de ejecución del Plugin, la API de extensiones y el SDK del Plugin ahora exponen
  ayudantes de filas de sesión respaldados por SQLite en lugar de ayudantes de compatibilidad
  de archivo/almacén completo de sesiones activas. Las exportaciones de compatibilidad de la biblioteca raíz siguen disponibles
  solo fuera del SDK del Plugin para llamadores internos heredados y de migración. El antiguo
  ayudante `resolveLegacySessionStorePath` se eliminó; la construcción de rutas `sessions.json`
  heredadas ahora es local de las migraciones y los accesorios de prueba.
- `src/config/sessions/session-entries.sqlite.ts` ahora almacena entradas de sesión
  canónicas en la base de datos por agente y admite parches de lectura/inserción o actualización/eliminación
  a nivel de fila. La inserción o actualización, el parcheado y la eliminación en tiempo de ejecución ya no buscan variantes de mayúsculas y minúsculas ni
  purgan claves de alias heredadas; el doctor se encarga de la canonicalización. Se eliminó el
  ayudante independiente de importación de JSON y la migración inserta o actualiza las filas más recientes
  al combinar, en lugar de reemplazar toda la tabla de sesiones. Los ayudantes públicos de lectura/listado/carga
  proyectan metadatos de sesión de acceso frecuente desde filas `sessions` y `conversations` con tipos;
  `entry_json` es una copia paralela de compatibilidad/depuración y puede estar obsoleta o no ser válida
  sin perder la identidad de sesión con tipos ni el contexto de entrega.
- `src/config/sessions/delivery-info.ts` ahora resuelve el contexto de entrega a partir de las
  filas con tipos `sessions` + `conversations` + `session_conversations` por agente.
  Ya no reconstruye la identidad de entrega en tiempo de ejecución a partir de
  `session_entries.entry_json`; la ausencia de una fila de conversación con tipos es un problema
  de migración/reparación del doctor, no una alternativa en tiempo de ejecución.
- Las decisiones de restablecimiento de sesiones almacenadas ahora priorizan los metadatos con tipos `sessions.session_scope`,
  `sessions.chat_type` y `sessions.channel`. El análisis de `sessionKey`
  se mantiene solo para sufijos explícitos de hilo/tema en destinos de comandos; la clasificación
  de restablecimiento grupal frente a directo ya no procede de la forma de la clave.
- La clasificación de visualización de listas/estados de sesiones ahora usa metadatos de chat con tipos y
  el tipo de sesión del Gateway. Ya no considera las subcadenas `:group:` o `:channel:`
  dentro de `session_key` como una fuente duradera de verdad sobre si es grupal o directo.
- La selección de la política de respuesta silenciosa ahora usa únicamente el tipo de conversación explícito o los metadatos
  de la superficie. Ya no infiere la política directa/grupal a partir de
  subcadenas de `session_key`.
- La resolución del modelo de visualización de sesión ahora recibe el id. del agente del destino de la base de datos
  de sesiones SQLite, en lugar de extraerlo dividiendo `session_key`.
- La hidratación del destino de anuncios entre agentes ahora usa únicamente `sessions.list`
  `deliveryContext` con tipos. Ya no recupera el enrutamiento de canal/cuenta/hilo
  de `origin` heredado, campos `last*` replicados ni la forma de `session_key`.
- El rechazo de destinos de hilo de `sessions_send` ahora lee metadatos de enrutamiento
  de SQLite con tipos. Ya no rechaza ni acepta destinos analizando sufijos de hilo
  extraídos de la clave de destino.
- La validación de políticas de herramientas con ámbito de grupo ahora lee el enrutamiento de conversación
  de SQLite con tipos para la sesión actual o generada. Ya no confía en la identidad de grupo/canal
  mediante la decodificación de `sessionKey`; los id. de grupo proporcionados por el llamador se descartan cuando
  ninguna fila de sesión con tipos los respalda.
- La coincidencia de anulaciones de modelo de canal ahora usa metadatos explícitos de la conversación
  grupal y principal. Ya no decodifica los id. de conversaciones principales a partir de
  `parentSessionKey`.
- La herencia de anulaciones de modelos almacenadas ahora requiere una clave de sesión principal explícita
  del contexto de sesión con tipos. Ya no deriva anulaciones principales de
  sufijos `:thread:` o `:topic:` en `sessionKey`.
- Se eliminaron el antiguo contenedor de información de hilos de sesión y el analizador de hilos de Plugins cargados;
  ningún código de tiempo de ejecución importa `config/sessions/thread-info`.
- El ayudante de conversación de canal ya no expone puentes de análisis
  de claves de sesión completas. El núcleo sigue normalizando los id. de conversación sin procesar propiedad del proveedor mediante
  `resolveSessionConversation(...)`, pero no reconstruye datos de ruta
  a partir de `sessionKey`.
- La entrega de finalización, la política de envío y el mantenimiento de tareas ya no derivan el tipo
  de chat de la forma de `session_key`. Se eliminó el antiguo analizador de claves de tipo de chat;
  estas rutas requieren metadatos de sesión con tipos, contexto de entrega con tipos o
  vocabulario explícito de destinos de entrega.
- Las listas/estados de sesiones, los diagnósticos, la vinculación de cuentas para aprobaciones, el filtrado de Heartbeat
  de la TUI y los resúmenes de uso ya no extraen de `SessionEntry.origin`
  el enrutamiento de proveedor/cuenta/hilo/visualización. Las únicas lecturas de `origin`
  restantes en tiempo de ejecución corresponden a conceptos ajenos a las sesiones u objetos de entrega del turno actual.
- La búsqueda de conversaciones nativas para solicitudes de aprobación ahora lee filas de enrutamiento de sesiones
  por agente con tipos. Ya no analiza la identidad de conversación de canal/grupo/hilo
  a partir de `sessionKey`; la ausencia de metadatos con tipos es un problema de migración/reparación.
- Las cargas útiles de eventos de cambio de sesión/chat/sesión del Gateway ya no replican
  las copias paralelas de ruta `SessionEntry.origin` ni `last*`; los clientes reciben
  `channel`, `chatType` y `deliveryContext` con tipos.
- La resolución de entrega de Heartbeat ahora puede recibir directamente el
  `deliveryContext` de SQLite con tipos, y el tiempo de ejecución de Heartbeat pasa la fila de entrega
  de sesión por agente en lugar de depender de copias paralelas de compatibilidad `session_entries`
  para el enrutamiento actual.
- La resolución del destino de entrega del agente aislado de Cron también hidrata su ruta
  actual desde la fila de entrega de sesión por agente con tipos antes de recurrir a la
  carga útil de entrada de compatibilidad.
- La resolución del origen de anuncios de subagentes ahora propaga el contexto de entrega
  de la sesión solicitante con tipos mediante `loadRequesterSessionEntry` y prioriza esa fila sobre
  las copias paralelas de compatibilidad `last*`/`deliveryContext`.
- Las actualizaciones de metadatos de sesión de entrada ahora se combinan primero con la fila de entrega
  por agente con tipos; los antiguos campos de entrega `SessionEntry` son solo la alternativa
  cuando no existe una fila de conversación con tipos.
- La extracción de entrega de reinicio/actualización ahora da prioridad a la entrega de SQLite
  `threadId` con tipos sobre los fragmentos de tema/hilo analizados de `sessionKey`; el análisis
  es solo una alternativa para claves heredadas con forma de hilo.
- Los id. de canal del contexto de agente de los hooks ahora priorizan la identidad de conversación de SQLite con tipos,
  seguida de los metadatos explícitos del mensaje. Ya no analizan fragmentos de proveedor/grupo/canal
  de `sessionKey`.
- La herencia de rutas externas de `chat.send` del Gateway ahora lee metadatos de enrutamiento de sesiones
  de SQLite con tipos en lugar de inferir el ámbito de canal/directo/grupo a partir de
  partes de `sessionKey`. Las sesiones con ámbito de canal solo heredan cuando el canal de sesión
  con tipos y el tipo de chat coinciden con el contexto de entrega almacenado; las sesiones principales
  compartidas mantienen su regla más estricta de CLI/sin metadatos del cliente.
- El despertar mediante centinela de reinicio y el enrutamiento de continuaciones ahora leen filas de
  entrega/enrutamiento de SQLite con tipos antes de poner en cola despertares de Heartbeat o continuaciones
  enrutadas de turnos del agente. Ya no reconstruyen el contexto de entrega a partir de la
  copia paralela JSON de la entrada de sesión.
- La resolución del contexto de `tools.effective` del Gateway ahora lee filas de
  entrega/enrutamiento de SQLite con tipos para las entradas de proveedor, cuenta, destino, hilo y modo
  de respuesta. Ya no recupera esos campos de enrutamiento de acceso frecuente de copias paralelas de origen
  `session_entries.entry_json` obsoletas.
- El enrutamiento de consultas de voz en tiempo real ahora resuelve la entrega principal/de llamada a partir de filas
  de sesión SQLite por agente con tipos. Ya no recurre a copias paralelas
  de compatibilidad `SessionEntry.deliveryContext` al elegir la ruta de mensajes
  del agente integrado.
- El relé de Heartbeat de generación de ACP y el enrutamiento de flujos principales ahora leen la entrega principal
  de filas de sesión SQLite con tipos. Ya no reconstruyen el contexto de entrega principal
  a partir de copias paralelas de compatibilidad de entradas de sesión.
- La conservación de rutas de entrega de sesiones ahora sigue los metadatos de chat con tipos y
  las columnas de entrega persistentes. Ya no extrae indicios de canal, marcadores
  de directo/principal ni la forma del hilo de `sessionKey`; las rutas internas de chat web solo
  heredan un destino externo cuando SQLite ya contiene una identidad de entrega
  con tipos/persistente para la sesión.
- La extracción genérica de entrega de sesiones ahora lee únicamente la fila exacta de entrega
  de sesión SQLite con tipos. Ya no analiza sufijos de hilo/tema ni recurre
  de una clave con forma de hilo a una clave de sesión base.
- El despacho de respuestas, la recuperación del centinela de reinicio y el enrutamiento de consultas de voz en tiempo real
  ahora usan filas exactas de sesión/conversación de SQLite con tipos para el enrutamiento de hilos. Ya
  no recuperan los id. de hilo ni el contexto de entrega de la sesión base mediante el análisis de
  claves de sesión con forma de hilo.
- La limitación del historial de PI integrado ahora usa la proyección de enrutamiento
  de sesiones SQLite con tipos (`sessions` + `conversations` principal) para el proveedor, el tipo de chat
  y la identidad del interlocutor. Ya no analiza la forma del proveedor, mensaje directo, grupo o hilo
  a partir de `sessionKey`.
- La inferencia de entrega de herramientas de Cron ahora usa únicamente la entrega explícita o el contexto de entrega
  actual con tipos. Ya no decodifica destinos de canal, interlocutor, cuenta o hilo
  a partir de `agentSessionKey`.
- Las filas de sesión en tiempo de ejecución ya no contienen el antiguo alias de ruta `lastProvider`.
  Los ayudantes y las pruebas usan los campos con tipos `lastChannel` y `deliveryContext`;
  la migración del doctor es el único lugar donde deben traducirse los alias de ruta antiguos
  o las copias paralelas `origin` persistentes.
- Los eventos de transcripción, las filas de VFS y las filas de artefactos de herramientas ahora se escriben en la base de datos
  por agente. Se eliminó la tabla global no publicada de asignación de archivos de transcripción; el doctor
  registra en su lugar las rutas de origen heredadas en filas de migración duraderas.
- La búsqueda de transcripciones en tiempo de ejecución ya no examina desplazamientos de bytes de JSONL ni sondea archivos de transcripción
  heredados. Las rutas de chat/multimedia/historial del Gateway leen filas de transcripción desde
  SQLite; el JSONL de sesión ahora es solo una entrada heredada del doctor, no un formato de estado
  ni de exportación en tiempo de ejecución.
- Las relaciones principales y de ramas de las transcripciones usan metadatos estructurados
  `parentTranscriptScope: {agentId, sessionId}` en las cabeceras de transcripciones de SQLite,
  no cadenas de localización `agent-db:...transcript_events...` similares a rutas.
- El contrato del gestor de transcripciones ya no expone constructores persistentes implícitos
  `create(cwd)` ni `continueRecent(cwd)`. Los gestores de transcripciones
  persistentes se abren con un ámbito `{agentId, sessionId}` explícito; solo
  Los gestores en memoria siguen sin ámbito para las pruebas y las transformaciones puras de transcripciones.
- Las API del almacén de transcripciones en tiempo de ejecución resuelven el ámbito de SQLite, no rutas del sistema de archivos. El
  antiguo asistente `resolve...ForPath` y las opciones de escritura `transcriptPath` sin usar han
  desaparecido de los llamadores en tiempo de ejecución.
- La resolución de sesiones en tiempo de ejecución ahora usa `{agentId, sessionId}` y no debe derivar
  cadenas `sqlite-transcript://<agent>/<session>` para límites externos.
  Las rutas JSONL absolutas heredadas son únicamente entradas para la migración de doctor.
- Los registros de puente directo de retransmisión de hooks nativos ahora residen en filas compartidas con tipos
  `native_hook_relay_bridges`, cuya clave es el id de retransmisión. El tiempo de ejecución ya no escribe un
  registro JSON `/tmp` ni registros genéricos opacos para esos registros de puente
  de corta duración.
- `runEmbeddedPiAgent(...)` ya no tiene un parámetro de localizador de transcripción.
  Los descriptores de workers preparados también omiten los localizadores de transcripciones. El estado
  de sesión en tiempo de ejecución y las ejecuciones de seguimiento en cola llevan `{agentId, sessionId}` en lugar de
  identificadores de transcripción derivados.
- La Compaction integrada ahora obtiene el ámbito de SQLite de `agentId` y `sessionId`.
  Los hooks de Compaction, las llamadas al motor de contexto, la delegación de la CLI y las respuestas del protocolo
  no deben recibir identificadores `sqlite-transcript://...` derivados. El código de exportación/depuración
  puede materializar artefactos de usuario explícitos a partir de filas, pero no proporciona una
  ruta genérica de exportación JSONL de sesiones ni devuelve nombres de archivo a la identidad
  en tiempo de ejecución.
- `/export-session` lee filas de transcripciones desde SQLite y escribe únicamente la vista HTML
  independiente solicitada. El visor integrado ya no reconstruye ni
  descarga el JSONL de la sesión a partir de esas filas.
- La delegación del motor de contexto ya no analiza un localizador de transcripción para recuperar
  la identidad del agente. El contexto preparado en tiempo de ejecución lleva el `agentId` resuelto
  al adaptador de Compaction integrado.
- La reescritura de transcripciones y el truncamiento de resultados de herramientas en vivo ahora leen y conservan
  el estado de la transcripción mediante `{agentId, sessionId}` y no derivan localizadores
  temporales para las cargas útiles de eventos de actualización de transcripciones.
- La superficie de asistentes de estado de transcripciones ya no tiene las variantes basadas en localizadores
  `readTranscriptState`, `replaceTranscriptStateEvents` ni
  `persistTranscriptStateMutation`. Los llamadores en tiempo de ejecución deben usar las API
  `{agentId, sessionId}`. La importación de doctor lee archivos heredados mediante una ruta de archivo
  explícita y escribe filas de SQLite; no migra cadenas de localizadores.
- El contrato del gestor de sesiones en tiempo de ejecución ya no expone `open(locator)`,
  `forkFrom(locator)` ni `setTranscriptLocator(...)`. Los gestores de sesiones
  persistentes se abren únicamente mediante `{agentId, sessionId}`; los asistentes de listado/bifurcación residen en
  las API de sesiones y puntos de control orientadas a filas, en lugar de en la fachada del gestor
  de transcripciones.
- Las API de lectura de transcripciones del Gateway priorizan el ámbito. Reciben
  `{agentId, sessionId}` y no aceptan un localizador de transcripción posicional que
  pudiera convertirse accidentalmente en la identidad en tiempo de ejecución. Se ha eliminado el análisis de localizadores
  de transcripciones activas; las rutas de origen heredadas solo las lee el código de importación de doctor.
- Los eventos de actualización de transcripciones también priorizan el ámbito. `emitSessionTranscriptUpdate`
  ya no acepta una cadena de localizador aislada, y los listeners enrutan mediante
  `{agentId, sessionId}` sin analizar un identificador.
- La difusión de mensajes de sesión del Gateway resuelve las claves de sesión a partir del ámbito
  del agente/de la sesión, no de un localizador de transcripción. Se ha eliminado el antiguo
  resolutor/caché de claves de sesión a partir de localizadores de transcripciones.
- El SSE del historial de sesiones del Gateway filtra las actualizaciones en vivo por el ámbito del agente/de la sesión. Ya no
  canoniza candidatos a localizadores de transcripciones, rutas reales ni identidades de transcripción
  con forma de archivo para decidir si un flujo debe recibir una actualización.
- Los hooks del ciclo de vida de las sesiones ya no derivan ni exponen localizadores de transcripciones en
  `session_end`. Los consumidores de hooks reciben `sessionId`, `sessionKey`, los ids de las
  sesiones siguientes y el contexto del agente; los archivos de transcripciones no forman parte del contrato
  del ciclo de vida.
- Los hooks de restablecimiento tampoco derivan ni exponen localizadores de transcripciones. La carga útil
  `before_reset` lleva los mensajes recuperados de SQLite junto con el motivo del restablecimiento,
  mientras que la identidad de la sesión permanece en el contexto del hook.
- El restablecimiento del arnés del agente ya no acepta un localizador de transcripción. El envío del restablecimiento se
  delimita mediante `sessionId`/`sessionKey` junto con el motivo.
- Los tipos de sesión de las extensiones de agentes ya no exponen `transcriptLocator`; las extensiones
  deben usar el contexto de sesión y las API en tiempo de ejecución en lugar de acceder a una
  identidad de transcripción con forma de archivo.
- Los hooks de Compaction de los plugins ya no exponen localizadores de transcripciones. El contexto del hook
  ya contiene la identidad de la sesión, y las lecturas de transcripciones deben realizarse mediante API
  compatibles con el ámbito de SQLite en lugar de identificadores con forma de archivo.
- Los hooks `before_agent_finalize` ya no exponen `transcriptPath`, incluidas
  las cargas útiles de retransmisión de hooks nativos. Los hooks de finalización solo usan el contexto de sesión.
- Las respuestas de restablecimiento del Gateway ya no sintetizan un localizador de transcripción en la
  entrada devuelta. El restablecimiento crea filas de transcripciones de SQLite, devuelve la entrada
  de sesión limpia y deja el acceso a las transcripciones en manos de lectores compatibles con el ámbito.
- Los resultados de ejecución integrada y Compaction ya no muestran localizadores de transcripciones para
  la contabilización de sesiones. La Compaction automática solo actualiza el `sessionId` activo,
  los contadores de Compaction y los metadatos de tokens.
- Los resultados de los intentos integrados ya no devuelven `transcriptLocatorUsed`, y
  los resultados `compact()` del motor de contexto ya no devuelven localizadores de transcripciones.
  Los bucles de reintento en tiempo de ejecución solo aceptan un `sessionId` sucesor.
- Los resultados de anexado de transcripciones del espejo de entrega ya no devuelven localizadores de
  transcripciones. Los llamadores reciben el `messageId` anexado; las señales de actualización de transcripciones usan
  el ámbito de SQLite.
- Los asistentes de bifurcación de sesiones principales devuelven únicamente el `sessionId` bifurcado. La preparación
  de subagentes pasa el ámbito del agente secundario/de la sesión a los motores.
- Los parámetros del ejecutor de la CLI y la reinicialización del historial ya no aceptan localizadores de transcripciones.
  Las lecturas del historial de la CLI resuelven el ámbito de la transcripción de SQLite a partir de `{agentId,
sessionId}` y del contexto de la clave de sesión.
- Los fixtures de prueba de la CLI y del ejecutor integrado ahora inicializan y leen filas de transcripciones de SQLite
  mediante el id de sesión, en lugar de fingir que las sesiones activas son archivos `*.jsonl` o
  pasar una cadena `sqlite-transcript://...` mediante los parámetros en tiempo de ejecución.
- Los eventos de protección de resultados de herramientas de sesión se emiten desde el ámbito de sesión conocido, incluso cuando un
  gestor en memoria no tiene un localizador derivado. Sus pruebas ya no simulan archivos de transcripciones
  `/tmp/*.jsonl` activos.
- Los asistentes BTW y de puntos de control de Compaction ahora leen y bifurcan filas de transcripciones mediante el
  ámbito de SQLite. Los metadatos de los puntos de control ahora almacenan únicamente ids de sesiones e ids
  de hojas/entradas; los localizadores derivados ya no se escriben en las cargas útiles de puntos de control.
- La búsqueda de claves de transcripciones del Gateway usa el ámbito de transcripciones de SQLite en los límites
  del protocolo y ya no resuelve rutas reales ni consulta estadísticas de nombres de archivos de transcripciones.
- La rotación de transcripciones de la Compaction automática escribe las filas de transcripciones sucesoras
  directamente mediante el almacén de transcripciones de SQLite. Las filas de sesiones conservan únicamente la
  identidad de la sesión sucesora, no una ruta JSONL duradera ni un localizador persistente.
- La Compaction integrada del motor de contexto usa asistentes de rotación de transcripciones
  con nombres de SQLite. Las pruebas de rotación ya no construyen rutas JSONL sucesoras ni
  modelan las sesiones activas como archivos.
- La retención gestionada de imágenes salientes genera las claves de su caché de mensajes de transcripciones a partir de
  las estadísticas de transcripciones de SQLite, en lugar de llamadas de estadísticas del sistema de archivos.
- Se han eliminado los bloqueos de sesiones en tiempo de ejecución y el flujo independiente heredado de doctor
  `.jsonl.lock`.
- El barrel en tiempo de ejecución de Microsoft Teams y el SDK público de plugins ya no reexportan
  el antiguo asistente de bloqueo de archivos; las rutas de estado duradero de los plugins están respaldadas por SQLite.
- Se han eliminado la depuración de sesiones por antigüedad/cantidad y la limpieza explícita de sesiones.
  Doctor se encarga de la importación heredada; las sesiones obsoletas se restablecen o eliminan explícitamente.
- Las comprobaciones de integridad de doctor ya no cuentan un archivo JSONL heredado como una transcripción activa
  válida para una fila de sesión de SQLite. El estado de las transcripciones activas depende únicamente de SQLite;
  los archivos JSONL heredados se notifican como entradas de migración/limpieza de huérfanos.
- Doctor ya no trata `agents/<agent>/sessions/` como estado obligatorio
  en tiempo de ejecución. Solo examina ese directorio cuando ya existe, como entrada para la importación
  heredada o la limpieza de huérfanos.
- Las rutas de `sessions.resolve` del Gateway, parche/restablecimiento/Compaction de sesiones, generación de
  subagentes, interrupción rápida, metadatos de ACP, sesiones aisladas por Heartbeat y parcheo de la TUI
  ya no migran ni depuran claves de sesiones heredadas como efecto secundario del
  funcionamiento normal en tiempo de ejecución.
- La resolución de sesiones de comandos de la CLI ahora devuelve el `agentId` propietario en lugar de un
  `storePath`, y ya no copia filas heredadas de la sesión principal durante la resolución normal
  de `--to` o `--session-id`. La canonización de filas principales heredadas corresponde
  únicamente a doctor.
- La resolución de profundidad de subagentes en tiempo de ejecución ya no lee `sessions.json` ni almacenes
  de sesiones JSON5. Lee `session_entries` de SQLite mediante el id del agente, y los metadatos
  heredados de profundidad/sesión solo pueden entrar por la ruta de importación de doctor.
- Las anulaciones de sesiones de perfiles de autenticación se conservan mediante upserts directos de filas
  `{agentId, sessionKey}`, en lugar de cargar de forma diferida un tiempo de ejecución de almacén de sesiones con forma de archivo.
- El control detallado de respuestas automáticas y los asistentes de actualización de sesiones ahora leen/actualizan mediante upsert filas
  de sesiones de SQLite según la identidad de la sesión y ya no requieren una ruta de almacén heredada
  antes de modificar el estado persistente de las filas.
- Los asistentes de metadatos de sesiones de ejecución de comandos ahora usan nombres y rutas de módulos
  orientados a entradas; se ha eliminado la antigua superficie de asistentes de comandos `session-store`.
- La inicialización de encabezados de arranque y el refuerzo de límites de Compaction manual ahora modifican
  directamente las filas de transcripciones de SQLite. Los llamadores en tiempo de ejecución pasan la identidad de la sesión, no
  rutas `.jsonl` escribibles.
- La reproducción silenciosa de la rotación de sesiones copia los turnos recientes de usuario/asistente mediante
  `{agentId, sessionId}` desde las filas de transcripciones de SQLite. Ya no acepta
  localizadores de transcripciones de origen o destino.
- Las filas nuevas de sesiones en tiempo de ejecución ya no almacenan localizadores de transcripciones. Los llamadores usan
  `{agentId, sessionId}` directamente; los comandos de exportación/depuración pueden elegir los nombres de los archivos
  de salida cuando materializan las filas.
- Al iniciar una nueva sesión de transcripción persistente, ahora siempre se abren filas de SQLite por
  ámbito. El gestor de sesiones ya no reutiliza una ruta o un localizador de transcripción anterior
  de la era de los archivos como identidad de la nueva sesión.
- Las sesiones de transcripciones persistentes usan la API explícita
  `openTranscriptSessionManagerForSession({agentId, sessionId})`. Las antiguas fachadas estáticas
  `SessionManager.create/openForSession/list/forkFromSession` han desaparecido para que las pruebas y el código en tiempo de ejecución no puedan
  recrear accidentalmente el descubrimiento de sesiones de la era de los archivos.
- El tiempo de ejecución de los plugins ya no expone `api.runtime.agent.session.resolveTranscriptLocatorPath`;
  el código de los plugins usa asistentes de filas de SQLite y valores de ámbito.
- La superficie pública del SDK `session-store-runtime` ahora solo exporta asistentes de filas
  de sesiones y filas de transcripciones. Los asistentes específicos de esquema/ruta/transacción de SQLite
  residen en `sqlite-runtime`; los asistentes básicos de apertura/cierre/restablecimiento siguen siendo locales
  únicamente para las pruebas propias.
- Los clasificadores heredados de nombres de archivos de trayectorias/puntos de control `.jsonl` ahora residen en el
  módulo de archivos de sesiones heredadas de doctor. La validación de sesiones del núcleo ya no importa
  asistentes de artefactos de archivos para decidir los ids normales de sesiones de SQLite.
- Las ejecuciones bloqueantes de subagentes de Active Memory usan filas de transcripciones de SQLite en lugar de
  crear archivos `session.jsonl` temporales o persistentes bajo el estado del plugin. Se ha
  eliminado la antigua opción `transcriptDir`.
- La generación puntual de slugs y las ejecuciones del planificador del agente del sistema usan filas de transcripciones de SQLite
  en lugar de crear archivos `session.jsonl` temporales.
- `llm-task` las ejecuciones auxiliares y la extracción de compromisos ocultos también usan filas de transcripción de SQLite, por lo que estas sesiones auxiliares exclusivas del modelo ya no crean archivos temporales de transcripción JSON/JSONL.
- `TranscriptSessionManager` ahora es únicamente un ámbito de transcripción de SQLite abierto.
  El código de runtime lo abre con `openTranscriptSessionManagerForSession({agentId,
sessionId})`; los flujos para crear, ramificar, continuar, enumerar y bifurcar residen en sus
  auxiliares propietarios de filas de SQLite, en lugar de en fachadas estáticas del gestor.
  El código de doctor/importación/depuración gestiona archivos de origen heredados explícitos fuera del
  gestor de sesiones del runtime.
- Se eliminaron los métodos obsoletos de fachada `SessionManager.newSession()` y
  `SessionManager.createBranchedSession()`. Sus flujos de trabajo propietarios de SQLite crean las
  sesiones nuevas y los descendientes de transcripciones, en lugar de convertir un gestor ya abierto en una sesión
  persistente distinta mediante mutación.
- Las decisiones de bifurcación de la transcripción principal y la creación de bifurcaciones ya no aceptan
  `storePath` ni `sessionsDir`; usan el ámbito de transcripción de SQLite
  `{agentId, sessionId}` en lugar de metadatos retenidos de rutas del sistema de archivos.
- Memory-host ya no exporta auxiliares inoperantes de clasificación de transcripciones
  del directorio de sesiones; el filtrado de transcripciones ahora se deriva de los metadatos de filas de SQLite
  durante la construcción de entradas.
- Las pruebas de exportación de sesiones de Memory-host y QMD usan ámbitos de transcripción de SQLite. Las rutas
  `agents/<agentId>/sessions/*.jsonl` antiguas solo siguen cubiertas cuando una prueba
  demuestra intencionadamente la compatibilidad de doctor/importación/exportación.
- La inspección de sesiones sin procesar de QA-lab ahora usa `sessions.list` mediante el gateway
  en lugar de leer `agents/qa/sessions/sessions.json`; los comentarios de MSteams
  se anexan directamente a las transcripciones de SQLite sin inventar una ruta JSONL.
- Los turnos compartidos de canales entrantes ahora contienen `{agentId, sessionKey}` en lugar de un
  `storePath` heredado. Las rutas de registro de LINE, WhatsApp, Slack, Discord, Telegram, Matrix, Signal,
  iMessage, BlueBubbles, Feishu, Google Chat, IRC, Nextcloud Talk, Zalo,
  Zalo Personal, QA Channel, Microsoft Teams, Mattermost, Synology Chat, Tlon,
  Twitch y QQBot ahora leen los metadatos de la última actualización y registran
  las filas de sesiones entrantes mediante la identidad de SQLite.
- Se elimina la persistencia del localizador de transcripciones de las filas de sesiones activas.
  `resolveSessionTranscriptTarget` devuelve `agentId`, `sessionId` y metadatos
  opcionales del tema; doctor es el único código que importa nombres heredados de archivos de
  transcripción.
- Los encabezados de transcripción del runtime comienzan en la versión de SQLite `1`. Las actualizaciones de formas
  antiguas JSONL V1/V2/V3 solo residen en la importación de doctor y normalizan los encabezados importados a
  la versión actual de transcripciones de SQLite antes de almacenar las filas.
- La protección de enfoque prioritario en la base de datos ahora prohíbe `SessionManager.listAll` y
  `SessionManager.forkFromSession`; los flujos de trabajo de enumeración de sesiones y bifurcación/restauración
  deben permanecer en las API de SQLite basadas en filas/ámbitos.
- La protección también prohíbe los nombres de auxiliares heredados para analizar JSONL de transcripciones o reparar ramas activas
  fuera del código de doctor/importación, de modo que el runtime no pueda desarrollar una segunda ruta de migración de
  transcripciones heredadas.
- Las ejecuciones de PI integrado rechazan los identificadores de transcripción entrantes. Usan la identidad de SQLite
  `{agentId, sessionId}` antes de iniciar el worker y de nuevo antes de que el
  intento modifique el estado de la transcripción. Una entrada `/tmp/*.jsonl` obsoleta no puede seleccionar un
  destino de escritura del runtime.
- Los registros de trazas de caché, cargas útiles de Anthropic, flujos sin procesar y cronologías de diagnóstico
  ahora se escriben en filas tipadas de SQLite `diagnostic_events`. Los paquetes de estabilidad del Gateway
  ahora se escriben en filas tipadas de SQLite `diagnostic_stability_bundles`. Se eliminaron las antiguas
  rutas de sustitución JSONL `diagnostics.cacheTrace.filePath`, `OPENCLAW_CACHE_TRACE_FILE`,
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` y
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH`, y la captura normal de estabilidad ya no escribe archivos
  `logs/stability/*.json`.
- La persistencia de Cron ahora concilia filas de SQLite `cron_jobs` en lugar de
  eliminar y reinsertar toda la tabla de trabajos con cada guardado. Las escrituras posteriores de destinos de Plugin
  actualizan directamente las filas de cron coincidentes y mantienen el estado de cron del runtime en
  la misma transacción de la base de datos de estado.
- Los llamadores del runtime de Cron ahora usan una clave estable del almacén de cron de SQLite. Las rutas heredadas
  `cron.store` son únicamente entradas de importación de doctor; las rutas del Gateway de producción, mantenimiento de tareas,
  estado, historial de ejecuciones y escritura posterior del destino de Telegram usan
  `resolveCronStoreKey` y ya no normalizan la ruta de la clave. El estado de Cron ahora
  informa de `storeKey` en lugar del antiguo campo con forma de archivo `storePath`.
- La carga y la programación del runtime de Cron ya no normalizan formas heredadas de trabajos persistentes
  como `jobId`, `schedule.cron`, `atMs` numérico, booleanos de cadena o
  la ausencia de `sessionTarget`. La importación heredada de doctor se encarga de esas reparaciones antes
  de insertar las filas en SQLite.
- La generación de ACP ya no resuelve ni persiste rutas de archivos JSONL de transcripciones. La configuración de
  generación y vinculación de hilos persiste directamente la fila de sesión de SQLite y conserva el
  id. de sesión como identidad retenida de la transcripción.
- Las API de metadatos de sesiones de ACP ahora leen/enumeran/insertan o actualizan filas de SQLite por `agentId` y
  ya no exponen `storePath` como parte del contrato de entrada de sesión de ACP.
- La contabilización del uso de sesiones y la agregación del uso del Gateway ahora resuelven las transcripciones
  únicamente mediante `{agentId, sessionId}`. La caché de costes/uso y los resúmenes de sesiones
  descubiertas ya no sintetizan ni devuelven cadenas localizadoras de transcripciones.
- La anexión de chat del Gateway, la persistencia parcial por cancelación, `/sessions.send` y
  las escrituras de transcripciones multimedia del chat web se anexan directamente mediante el ámbito de transcripción
  de SQLite. El auxiliar de inyección de transcripciones del Gateway ya no acepta un
  parámetro `transcriptLocator`.
- El descubrimiento de transcripciones de SQLite ahora solo enumera ámbitos y estadísticas de transcripciones:
  `{agentId, sessionId, updatedAt, eventCount}`. Se eliminaron el auxiliar de
  compatibilidad inactivo `listSqliteSessionTranscriptLocators` y el campo por fila
  `locator`.
- El runtime de reparación de transcripciones ahora solo expone
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})`. Se eliminó el antiguo
  auxiliar de reparación basado en localizadores; el código de doctor/depuración lee rutas explícitas de
  archivos de origen y nunca migra cadenas localizadoras.
- El runtime del registro de reproducción de ACP ahora almacena filas de reproducción por sesión en la base de datos
  de estado compartida de SQLite en lugar de `acp/event-ledger.json`; doctor importa y
  elimina el archivo heredado.
- Los auxiliares de lectura de transcripciones del Gateway ahora residen en
  `src/gateway/session-transcript-readers.ts` en lugar del antiguo nombre de módulo
  `session-utils.fs`. La comprobación del historial de reintentos alternativos recibe un nombre basado en
  el contenido de las transcripciones de SQLite, en lugar de en la antigua superficie del auxiliar de archivos.
- Los auxiliares de chat inyectado y Compaction del Gateway ahora pasan el ámbito de transcripción de SQLite
  mediante API auxiliares internas, en lugar de denominar a los valores rutas de transcripción o
  archivos de origen.
- La detección de continuación de bootstrap ahora comprueba las filas de transcripción de SQLite mediante
  `hasCompletedBootstrapTranscriptTurn`; ya no expone un nombre de
  auxiliar con forma de archivo.
- Las pruebas del ejecutor integrado ahora usan la identidad de transcripción de SQLite, y abrir un nuevo
  gestor de transcripciones siempre requiere un `sessionId` explícito.
- Los auxiliares de indexación de memoria ahora usan terminología de transcripciones de SQLite de principio a fin:
  el host exporta `listSessionTranscriptScopesForAgent` y
  `sessionTranscriptKeyForScope`, la sincronización dirigida pone en cola `sessionTranscripts`,
  los resultados públicos de búsqueda de sesiones exponen rutas opacas `transcript:<agent>:<session>`,
  y la clave interna de origen de la base de datos es `session:<session>` bajo
  `source_kind='sessions'`, en lugar de una ruta de archivo ficticia.
- El auxiliar genérico de deduplicación persistente del SDK de Plugin ya no expone opciones
  con forma de archivo. Los llamadores proporcionan claves de ámbito de SQLite y las filas duraderas de deduplicación residen en
  el estado compartido del Plugin.
- Los tokens SSO de Microsoft Teams se trasladaron de archivos JSON bloqueados al estado de Plugin de SQLite.
  Doctor importa `msteams-sso-tokens.json`, reconstruye las claves canónicas de tokens
  SSO a partir de las cargas útiles y elimina el archivo de origen. Los tokens OAuth delegados permanecen
  en su límite existente de archivos privados de credenciales.
- El estado de la caché de sincronización de Matrix se trasladó de `bot-storage.json` al estado de Plugin de SQLite.
  Doctor importa las cargas útiles heredadas de sincronización, sin procesar o encapsuladas, y elimina el
  archivo de origen. Los clientes activos del adaptador de Matrix y de Matrix de QA Lab pasan un directorio raíz del almacén de sincronización de SQLite,
  no una ruta ficticia `sync-store.json` o `bot-storage.json`.
- El estado de migración criptográfica heredada de Matrix se trasladó de
  `legacy-crypto-migration.json` al estado de Plugin de SQLite. Doctor importa el
  antiguo archivo de estado; las instantáneas de IndexedDB del SDK de Matrix se trasladaron de
  `crypto-idb-snapshot.json` a blobs de Plugin de SQLite. Las claves de recuperación y
  las credenciales de Matrix son filas del estado de Plugin de SQLite; sus antiguos archivos JSON son únicamente entradas de
  migración de doctor.
- Los registros de actividad de Memory Wiki ahora usan el estado de Plugin de SQLite en lugar de
  `.openclaw-wiki/log.jsonl`. El proveedor de migración de Memory Wiki importa los antiguos
  registros JSONL; el contenido Markdown de la wiki y del almacén del usuario permanece respaldado por archivos como
  contenido del espacio de trabajo.
- Memory Wiki ya no crea `.openclaw-wiki/state.json` ni el directorio sin uso
  `.openclaw-wiki/locks`. El proveedor de migración elimina esos
  archivos retirados de metadatos del Plugin si un almacén antiguo aún los contiene.
- Las entradas de auditoría del agente del sistema ahora usan el estado de Plugin de SQLite del núcleo en lugar de
  `audit/crestodian.jsonl`. Doctor importa el registro de auditoría JSONL heredado y
  lo elimina tras importarlo correctamente.
- Las entradas de auditoría de escritura/observación de configuración ahora usan el estado de Plugin de SQLite del núcleo en lugar de
  `logs/config-audit.jsonl`. Doctor importa el registro de auditoría JSONL heredado y
  lo elimina tras importarlo correctamente.
- La aplicación complementaria de macOS ya no escribe archivos auxiliares locales de la aplicación `logs/config-audit.jsonl` ni
  `logs/config-health.json` al editar `openclaw.json`. El archivo de configuración
  sigue respaldado por archivos, las instantáneas de recuperación permanecen junto al archivo de configuración
  y el estado duradero de auditoría/salud de la configuración pertenece al almacén SQLite del Gateway.
- Las aprobaciones pendientes de rescate del agente del sistema ahora usan el estado de Plugin de SQLite del núcleo en lugar de
  `crestodian/rescue-pending/*.json` o `openclaw/rescue-pending/*.json`.
  Estas capacidades de seguridad de corta duración nunca se importan; doctor descarta
  ambos directorios retirados para que una actualización no pueda reactivar una escritura obsoleta.
- El estado de armado temporal de Phone Control ahora usa el estado de Plugin de SQLite en lugar de
  `plugins/phone-control/armed.json`. Doctor importa el archivo heredado del estado armado
  en el espacio de nombres `phone-control/arm-state` y elimina el archivo.
- Doctor ya no repara las transcripciones JSONL in situ ni crea archivos JSONL de
  copia de seguridad. Importa la rama activa en SQLite y elimina el origen heredado.
- La búsqueda de transcripciones del hook de memoria de sesión usa lecturas de SQLite exclusivas del ámbito
  `{agentId, sessionId}`. Su auxiliar ya no acepta ni deriva localizadores de transcripciones,
  lecturas de archivos heredados ni opciones de reescritura de archivos.
- Las vinculaciones de conversaciones del servidor de aplicaciones de Codex ahora asignan claves al estado de Plugin de SQLite mediante
  la clave de sesión de OpenClaw o un ámbito `{agentId, sessionId}` explícito. No deben
  conservar vinculaciones alternativas basadas en rutas de transcripciones.
- Las lecturas del historial reflejado del servidor de aplicaciones de Codex usan únicamente el ámbito de transcripción de SQLite;
  no deben recuperar la identidad a partir de rutas de archivos de transcripción.
- Las rutas de ordenación de roles y restablecimiento de Compaction ya no desvinculan archivos antiguos de
  transcripción; el restablecimiento solo rota la fila de sesión de SQLite y la identidad de la transcripción.
- Las respuestas de restablecimiento y punto de control del Gateway devuelven filas de sesión limpias junto con los
  id. de sesión. Ya no sintetizan localizadores de transcripciones de SQLite para los clientes.
- Dreaming de memory-core ya no elimina filas de sesiones mediante la comprobación de archivos
  JSONL ausentes. La limpieza de subagentes se realiza mediante la API del runtime de sesiones, en lugar de
  comprobaciones de existencia en el sistema de archivos. Sus pruebas de ingesta de transcripciones insertan directamente filas de SQLite
  en lugar de crear accesorios `agents/<id>/sessions` o marcadores de posición de
  localizadores.
- La indexación de transcripciones de memoria puede exponer `transcript:<agentId>:<sessionId>` como una
  ruta virtual de resultado de búsqueda para los auxiliares de cita/lectura. El origen duradero del índice es
  relacional (`source_kind='sessions'`, `source_key='session:<sessionId>'`,
  `session_id=<sessionId>`), por lo que el valor no es un localizador de transcripción en tiempo de ejecución,
  no es una ruta del sistema de archivos y nunca debe volver a pasarse a las API de tiempo de ejecución de sesiones.
- El estado de memoria de doctor del Gateway lee los recuentos de recuperación a corto plazo y señales de fase
  de las filas de estado del Plugin en SQLite en lugar de `memory/.dreams/*.json`; la CLI y
  la salida de doctor ahora identifican ese almacenamiento como un almacén SQLite, no como una ruta.
- El tiempo de ejecución de memory-core, el estado de la CLI, los métodos de doctor del Gateway y las fachadas
  del SDK de plugins ya no auditan ni archivan archivos heredados `.dreams/session-corpus`.
  Esos archivos son únicamente entradas de migración; doctor los importa a SQLite y
  elimina el origen después de verificarlo. Las filas de evidencia de ingesta de sesiones activas
  ahora usan la ruta virtual de SQLite `memory/session-ingestion/<day>.txt`; el tiempo de ejecución
  nunca escribe ni deriva estado de `.dreams/session-corpus`.
- Los artefactos públicos de memory-core exponen los eventos del host de SQLite como el artefacto JSON
  virtual `memory/events/memory-host-events.json`; ya no reutilizan la ruta de origen
  heredada `.dreams/events.jsonl`.
- Los registros de contenedores/navegadores del sandbox ahora usan la tabla SQLite
  compartida `sandbox_registry_entries` con columnas tipadas de sesión, imagen, marca de tiempo,
  backend/configuración y puerto del navegador. Doctor importa los archivos de registro JSON
  monolíticos y fragmentados heredados y elimina los orígenes procesados correctamente. Las lecturas en tiempo de ejecución usan
  las columnas tipadas de las filas como fuente de verdad; `entry_json` es solo una copia
  para reproducción/depuración.
- Los compromisos ahora usan una tabla compartida tipada `commitments` en lugar de un
  blob JSON de todo el almacén. El tiempo de ejecución usa consultas indexadas de ámbito, ventana de entrega, límite
  móvil, estado e intentos, además de transacciones SQLite síncronas;
  `record_json` es solo una copia para reproducción/depuración. La reparación explícita de doctor valida
  el `commitments.json` heredado completo, conserva las filas SQLite más recientes, verifica el
  resultado y solo entonces elimina el origen sin modificar. El tiempo de ejecución nunca lee ni
  escribe el archivo retirado.
- Las suscripciones de Web Push y la identidad VAPID generada ahora usan filas compartidas
  tipadas `web_push_subscriptions` y `web_push_vapid_keys`. El registro en tiempo de ejecución,
  la limpieza por caducidad y la generación de claves en el primer uso emplean transacciones SQLite
  a nivel de fila. La reparación explícita de Doctor valida ambos almacenes JSON retirados,
  los reclama antes de escribir en SQLite, los importa atómicamente, rechaza
  identidades VAPID en conflicto, verifica el resultado y solo entonces elimina las
  reclamaciones. Doctor mantiene el bloqueo de mantenimiento del directorio de estado durante toda la
  importación para que un Gateway antiguo no pueda volver a crear los archivos retirados. El registro,
  la entrega, la eliminación y la resolución de claves fallan de forma cerrada hasta que Doctor resuelva
  los orígenes heredados pendientes o las reclamaciones interrumpidas.
- Las definiciones de trabajos Cron, el estado de las programaciones y el historial de ejecuciones ya no tienen lectores
  ni escritores JSON en tiempo de ejecución. El tiempo de ejecución usa filas `cron_jobs` con columnas tipadas de programación,
  carga útil, entrega, alerta de fallo, sesión, estado y estado de ejecución, además de
  detalles `task_runs` propiedad de Cron para diagnósticos, entrega, sesión/ejecución, modelo
  y totales de tokens. `job_json` es solo una copia para reproducción/depuración; `state_json` conserva
  diagnósticos anidados del tiempo de ejecución que aún no tienen campos de consulta frecuentes, mientras que el tiempo de ejecución
  rehidrata los campos de estado frecuentes a partir de columnas tipadas. Doctor importa
  los archivos heredados `jobs.json`, `jobs-state.json` y `runs/*.jsonl` y elimina
  los orígenes importados. Las escrituras de retorno de destinos de plugins actualizan las filas `cron_jobs`
  correspondientes en lugar de cargar y sustituir todo el almacén de Cron.
- El inicio del Gateway ignora los marcadores heredados `notify: true` en la proyección
  del tiempo de ejecución. Doctor lee el `cron.webhook` sin procesar retirado solo mientras traduce
  esos marcadores en entregas SQLite explícitas y después elimina la clave de configuración.
- Las colas de entrega saliente y de sesiones ahora almacenan el estado de la cola, el tipo de entrada,
  la clave de sesión, el canal, el destino, el id. de cuenta, el número de reintentos, el último intento/error,
  el estado de recuperación y los marcadores de envío de plataforma como columnas tipadas en la tabla compartida
  `delivery_queue_entries`. La recuperación en tiempo de ejecución lee esos campos frecuentes de
  las columnas tipadas, y las mutaciones de reintento/recuperación actualizan directamente esas columnas
  sin reescribir el JSON de reproducción. La carga útil JSON completa permanece únicamente como
  blob de reproducción/depuración para los cuerpos de los mensajes y otros datos de reproducción de acceso poco frecuente.
- Los registros administrados de imágenes salientes ahora usan filas compartidas tipadas
  `managed_outgoing_image_records`. El tiempo de ejecución solo lee las columnas tipadas; la
  columna JSON es una copia para reproducción/depuración. Los bytes originales de las imágenes permanecen como
  artefactos de adjuntos con nombre en el directorio de medios administrados.
- Las preferencias del selector de modelos de Discord, los hashes de despliegue de comandos y las vinculaciones de hilos
  ahora usan el estado compartido de plugins en SQLite. Sus planes de importación de JSON heredado residen en la
  superficie de migración de configuración/doctor del Plugin de Discord, no en el código de migración del núcleo.
- Los detectores de importaciones heredadas de plugins usan módulos con nombres de doctor, como
  `doctor-legacy-state.ts` o `doctor-state-imports.ts`; los módulos normales de tiempo de ejecución
  de canales no deben importar detectores de JSON heredado.
- Los cursores de puesta al día y los marcadores de deduplicación entrante de BlueBubbles ahora usan el estado compartido
  de plugins en SQLite. Sus planes de importación de JSON heredado residen en la superficie de migración
  de configuración/doctor del Plugin de BlueBubbles, no en el código de migración del núcleo.
- Los desplazamientos de actualizaciones, las filas de caché de stickers, las filas de caché de mensajes enviados,
  las filas de caché de nombres de temas y las vinculaciones de hilos de Telegram ahora usan el estado compartido de plugins
  en SQLite. Sus planes de importación de JSON heredado residen en la superficie de migración
  de configuración/doctor del Plugin de Telegram, no en el código de migración del núcleo.
- Los cursores de puesta al día, las asignaciones de id. cortos de respuestas y las filas de deduplicación de ecos enviados
  de iMessage ahora usan el estado compartido de plugins en SQLite. Los archivos antiguos `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` y `imessage/sent-echoes.jsonl` son
  únicamente entradas de doctor.
- Las filas de deduplicación de mensajes de Feishu ahora utilizan la deduplicación reclamable del núcleo
  (espacios de nombres `feishu.dedup.*` en el estado compartido de plugins en SQLite) en lugar de
  archivos `feishu/dedup/*.json` o el almacén artesanal retirado `dedup.*`, sin
  importación heredada porque la caché de protección contra reproducciones se reconstruye después de la actualización.
- Las conversaciones, encuestas, búferes de carga pendientes y aprendizajes de comentarios de
  Microsoft Teams ahora usan tablas compartidas de estado/blobs de plugins en SQLite. La ruta de cargas pendientes
  usa `plugin_blob_entries` para que los búferes de medios se almacenen como BLOB de SQLite
  en lugar de JSON base64. Los nombres de los ayudantes del tiempo de ejecución ahora usan terminología de SQLite/estado
  en lugar de terminología de almacén de archivos `*-fs`, y la antigua capa de compatibilidad `storePath` ha desaparecido
  de estos almacenes. Su plan de importación de JSON heredado reside en la superficie de migración
  de configuración/doctor del Plugin de Microsoft Teams.
- Los medios salientes alojados de Zalo ahora usan `plugin_blob_entries` de SQLite compartido
  en lugar de archivos auxiliares temporales JSON/bin `openclaw-zalo-outbound-media`.
- El HTML y los metadatos del visor de diferencias ahora usan `plugin_blob_entries` de SQLite compartido
  en lugar de archivos temporales `meta.json`/`viewer.html`. El HTML del visor se almacena como un
  blob gzip y solo se conserva el hash del token de la URL. Las salidas PNG/PDF renderizadas
  permanecen como materializaciones temporales porque la entrega por canal aún necesita una ruta de archivo;
  sus metadatos de caducidad pertenecen a SQLite, sin archivos auxiliares JSON.
- Los documentos administrados de Canvas ahora usan `plugin_blob_entries` de SQLite compartido en lugar
  de un directorio predeterminado `state/canvas/documents`. El host de Canvas sirve esos
  blobs directamente; los archivos locales solo se crean para contenido explícito del operador `host.root`
  o para una materialización temporal cuando un lector de medios posterior
  requiere una ruta.
- Las decisiones de auditoría de File Transfer ahora usan `plugin_state_entries` de SQLite compartido
  en lugar del registro de tiempo de ejecución ilimitado `audit/file-transfer.jsonl`. Doctor
  importa el archivo de auditoría JSONL heredado al estado del Plugin y elimina el origen
  después de una importación limpia.
- Los arrendamientos de procesos y la identidad de instancia del Gateway de ACPX ahora usan el estado compartido de plugins
  en SQLite. Doctor importa el archivo heredado `gateway-instance-id` al estado del Plugin
  y elimina el origen.
- Los scripts envoltorio generados por ACPX y el directorio aislado de Codex son una
  materialización temporal bajo la raíz temporal de OpenClaw, no estado duradero de OpenClaw. Los
  registros duraderos del tiempo de ejecución de ACPX son las filas SQLite de arrendamiento e instancia del Gateway;
  la antigua superficie de configuración `stateDir` de ACPX se elimina porque ya no se escribe
  ningún estado del tiempo de ejecución allí.
- Los adjuntos de medios del Gateway ahora usan la tabla SQLite compartida `media_blobs` como
  almacén canónico de bytes. Las rutas locales devueltas a las superficies de compatibilidad
  de canales y del sandbox son materializaciones temporales de la fila de la base de datos, no el
  almacén duradero de medios. Las listas de permitidos de medios en tiempo de ejecución ya no incluyen las raíces heredadas
  `$OPENCLAW_STATE_DIR/media` ni `media` del directorio de configuración; esos directorios son
  únicamente orígenes de importación de doctor.
- La finalización del shell ya no escribe archivos de caché
  `$OPENCLAW_STATE_DIR/completions/*`. Las rutas de pruebas de humo de instalación, doctor, actualización y lanzamiento usan
  la salida de finalización generada o la carga desde el perfil en lugar de archivos duraderos de caché
  de finalización.
- El área de preparación de cargas de Skills del Gateway ahora usa filas compartidas `skill_uploads` y
  `skill_upload_chunks`. Cada fragmento permanece transaccional durante la
  carga; después, la confirmación ensambla un único BLOB de archivo verificado y elimina las filas de
  fragmentos. El instalador solo recibe una ruta temporal del archivo materializado mientras
  hay una instalación en curso. Doctor descarta el árbol retirado de preparación del sistema de archivos
  con duración de una hora en lugar de importar cargas transitorias.
- Los adjuntos en línea de los subagentes ya no se materializan bajo
  `.openclaw/attachments/*` del espacio de trabajo. La ruta de creación prepara entradas iniciales del VFS de SQLite,
  las ejecuciones en línea incorporan esas entradas al espacio de nombres temporal del tiempo de ejecución por agente,
  y las herramientas respaldadas por disco superponen ese espacio temporal de SQLite para las rutas de los adjuntos. Han desaparecido
  las antiguas columnas de registro de directorios de adjuntos de ejecución de subagentes y los enlaces de limpieza.
- La hidratación de imágenes de la CLI ya no mantiene archivos estables de caché
  `openclaw-cli-images`. Los backends externos de la CLI siguen recibiendo rutas de archivo, pero esas rutas son
  materializaciones temporales por ejecución con limpieza.
- Los diagnósticos de seguimiento de caché, los diagnósticos de cargas útiles de Anthropic, los diagnósticos de flujos
  sin procesar de modelos, los eventos de la cronología de diagnósticos y los paquetes de estabilidad del Gateway ahora
  escriben filas SQLite en lugar de archivos `logs/*.jsonl` o
  `logs/stability/*.json`.
  Se han eliminado las marcas y variables de entorno de sobrescritura de rutas en tiempo de ejecución; los comandos de exportación/depuración
  pueden materializar archivos explícitamente a partir de las filas de la base de datos.
- La aplicación complementaria de macOS ya no tiene un escritor rotativo de `diagnostics.jsonl`. Los registros de la
  aplicación van al registro unificado, y los diagnósticos duraderos del Gateway permanecen respaldados por SQLite.
- La lista de registros del guardián de puertos de macOS ahora usa filas compartidas tipadas de SQLite
  `macos_port_guardian_records` en lugar de un archivo JSON de Application Support
  o un blob singleton opaco. Todos los perfiles de la aplicación de macOS usan la misma base de datos nativa
  global del host porque coordinan puertos locales de la máquina. Cada operación del libro mayor
  se bloquea mientras se ejecuta una copia antigua de la aplicación que escribe JSON. La migración se incorpora al
  protocolo estable de bloqueo de archivos del libro mayor antiguo únicamente para obtener una instantánea y posteriormente revalidar el
  origen. Resuelve cada fila heredada a partir de datos actuales de comandos e inicio de procesos
  sin mantener ese bloqueo; después vuelve a leer las filas SQLite autoritativas, aplica el
  plan, verifica cada recibo y elimina el origen. Los reintentos de eliminación vuelven a planificar
  las filas ausentes para que los recibos obsoletos retirados no puedan reaparecer. El bloqueo permanece
  activo poco tiempo para que no pueda dejar bloqueado a un escritor antiguo después de que SSH haya iniciado un proceso. La transición es
  deliberadamente unidireccional: el tiempo de ejecución estable nunca lee, proyecta ni escribe JSON,
  y volver a compilaciones que solo usan JSON no conserva los recibos SQLite más recientes.
- Los bloqueos singleton del Gateway ahora usan filas compartidas tipadas de SQLite `state_leases` bajo
  el ámbito `gateway_locks` en lugar de archivos de bloqueo del directorio temporal. La documentación de solución de problemas
  de Fly y OAuth ahora remite al bloqueo de arrendamiento/actualización de autenticación de SQLite en lugar de
  a la limpieza obsoleta de bloqueos de archivos.
- El estado del centinela de reinicio del Gateway ahora usa filas SQLite compartidas y tipadas
  `gateway_restart_sentinel` en lugar de `restart-sentinel.json`; el entorno de ejecución
  lee el tipo, el estado, el enrutamiento, el mensaje, la continuación y las estadísticas del centinela desde
  columnas tipadas. Esas columnas son la fuente autoritativa; `payload_json` es solo una
  copia secundaria para reproducción y depuración. Las rutas de lectura, escritura y borrado del entorno de ejecución usan exclusivamente SQLite.
  Un módulo acotado de migración de estado se ejecuta durante el inicio y Doctor para importar un
  centinela anterior posterior a la actualización que se haya validado antes de la recuperación normal del reinicio, verificar
  la fila tipada y eliminar el archivo de origen. Ningún módulo del entorno de ejecución en estado estable
  lee, escribe ni limpia el archivo heredado.
- La intención de reinicio del Gateway y el estado de traspaso al supervisor ahora usan filas SQLite compartidas y tipadas
  `gateway_restart_intent` y `gateway_restart_handoff` en lugar de los archivos auxiliares
  `gateway-restart-intent.json` y
  `gateway-supervisor-restart-handoff.json`.
- La coordinación de instancia única del Gateway ahora usa filas tipadas `state_leases` bajo
  `gateway_locks` en lugar de escribir archivos `gateway.<hash>.lock`. La fila de concesión
  contiene el propietario del bloqueo, el vencimiento, el heartbeat y la carga útil de depuración; SQLite controla el
  límite atómico de adquisición y liberación. La opción retirada del directorio de bloqueos de archivos
  se eliminó; las pruebas usan directamente la identidad de la fila SQLite.
- Se eliminó el antiguo ayudante sin referencias para informes de uso de Cron que examinaba archivos `cron/runs/*.jsonl`.
  Los informes del historial de ejecuciones de Cron leen filas `task_runs` propiedad de Cron.
- La recuperación tras reinicios de la sesión principal ahora descubre agentes candidatos mediante el
  registro SQLite `agent_databases` en lugar de examinar directorios `agents/*/sessions`.
- La recuperación tras daños en sesiones de Gemini ahora elimina únicamente la fila de sesión de SQLite;
  ya no necesita una condición heredada `storePath` ni intenta desvincular una ruta JSONL
  derivada de la transcripción.
- La gestión de anulaciones de rutas ahora trata los valores literales de entorno `undefined`/`null`
  como no definidos, lo que evita bases de datos `undefined/state/*.sqlite` accidentales en la raíz del repositorio
  durante las pruebas o los traspasos de shell.
- Las huellas digitales de estado de la configuración ahora usan filas SQLite compartidas y tipadas `config_health_entries`
  en lugar de `logs/config-health.json`, lo que mantiene el archivo de configuración normal como
  el único documento de configuración que no contiene credenciales. La aplicación complementaria de macOS conserva únicamente
  el estado de funcionamiento local del proceso y no vuelve a crear el antiguo archivo auxiliar JSON.
- El entorno de ejecución de perfiles de autenticación ya no importa ni escribe archivos JSON de credenciales. El
  almacén canónico de credenciales es SQLite; `auth-profiles.json`, el archivo por agente
  `auth.json` y el archivo compartido `credentials/oauth.json` son entradas de migración de Doctor
  que se eliminan después de importarlas.
- Las pruebas de guardado y estado de perfiles de autenticación ahora verifican directamente las tablas tipadas de autenticación de SQLite
  y solo usan nombres de archivos heredados de perfiles de autenticación como entradas de migración de Doctor.
- `openclaw secrets apply` depura únicamente el archivo de configuración, el archivo de entorno y el almacén SQLite
  de perfiles de autenticación. Ya no incluye lógica de compatibilidad que edite
  el archivo retirado por agente `auth.json`; Doctor se encarga de importar y eliminar ese archivo.
- Los planes de migración de secretos de Hermes importan y aplican perfiles de claves de API directamente
  en el almacén SQLite de perfiles de autenticación. Ya no escriben ni verifican
  `auth-profiles.json` como destino intermedio.
- La documentación de autenticación dirigida a usuarios ahora describe
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>` en lugar de
  indicar a los usuarios que inspeccionen o copien `auth-profiles.json`; los nombres heredados de JSON
  de OAuth y autenticación solo se siguen documentando como entradas de importación de Doctor.
- Las sesiones OAuth de MCP ahora usan filas versionadas `mcp_oauth_stores` en el almacén compartido
  `state/openclaw.sqlite`. Los objetos de token, registro de cliente y descubrimiento
  propiedad del SDK permanecen en una única carga útil JSON validada para conservar los campos de extensión
  de dependencias, mientras que cada lectura, modificación y escritura se confirma en una transacción Kysely
  breve. Una única concesión SQLite compartida serializa la actualización, el inicio de sesión y el cierre de sesión;
  los transportes MCP integrados ya no permiten que el SDK de MCP actualice fuera de esa
  concesión. Doctor importa y elimina exclusivamente los almacenes retirados `mcp-oauth/*.json`
  con comprobantes de origen, y el entorno de ejecución no tiene respaldo mediante archivos.
- Los ayudantes de rutas de estado del núcleo ya no exponen el archivo retirado `credentials/oauth.json`.
  El nombre de archivo heredado es local a la ruta de importación de autenticación de Doctor.
- La documentación de instalación, seguridad, incorporación, autenticación de modelos y SecretRef ahora describe
  filas SQLite de perfiles de autenticación y la copia de seguridad y migración del estado completo en lugar de
  archivos JSON de perfiles de autenticación por agente.
- El descubrimiento de modelos de PI ahora pasa las credenciales canónicas al almacenamiento de autenticación
  en memoria `pi-coding-agent`. Ya no crea, depura ni escribe
  `auth.json` por agente durante el descubrimiento.
- Los ajustes de activación y enrutamiento de Voice Wake ahora usan tablas SQLite compartidas y tipadas
  en lugar de `settings/voicewake.json`, `settings/voicewake-routing.json` o
  filas genéricas opacas; Doctor importa los archivos JSON heredados y los elimina después de una
  migración correcta.
- El estado de comprobación de actualizaciones ahora usa una fila compartida y tipada `update_check_state` en lugar de
  `update-check.json` o un blob genérico opaco; Doctor importa
  el archivo JSON heredado y lo elimina después de una migración correcta.
- El estado de funcionamiento de la configuración ahora usa filas compartidas y tipadas `config_health_entries` en lugar
  de `logs/config-health.json` o un blob genérico opaco; Doctor
  importa el archivo JSON heredado y lo elimina después de una migración correcta.
- Las aprobaciones de vinculaciones de conversaciones de plugins ahora usan filas tipadas
  `plugin_binding_approvals` en lugar de estado SQLite compartido opaco o
  `plugin-binding-approvals.json`; el archivo heredado es una entrada de migración de Doctor.
- Las vinculaciones genéricas de la conversación actual ahora almacenan filas tipadas
  `current_conversation_bindings` en lugar de reescribir
  `bindings/current-conversations.json`; Doctor importa el archivo JSON heredado y
  lo elimina después de una migración correcta.
- Los registros de sincronización de fuentes importadas de Memory Wiki ahora almacenan una fila de estado de plugin SQLite
  por clave de bóveda y fuente en lugar de reescribir `.openclaw-wiki/source-sync.json`;
  el proveedor de migración importa y elimina el registro JSON heredado.
- Los registros de ejecuciones de importación de ChatGPT de Memory Wiki ahora almacenan una fila de estado de plugin SQLite
  por identificador de bóveda y ejecución en lugar de escribir `.openclaw-wiki/import-runs/*.json`.
  Las instantáneas de reversión siguen siendo archivos explícitos de la bóveda hasta que el archivado de instantáneas
  de ejecuciones de importación se traslade al almacenamiento de blobs.
- Los resúmenes compilados de Memory Wiki ahora almacenan filas comprimidas de blobs de plugin SQLite
  en lugar de escribir `.openclaw-wiki/cache/agent-digest.json` y
  `.openclaw-wiki/cache/claims.jsonl`. La caché se puede reconstruir, por lo que Doctor
  elimina los archivos de caché antiguos sin importarlos.
- El seguimiento de instalaciones de Skills de ClawHub ahora almacena una fila de estado de plugin SQLite por
  espacio de trabajo y Skill en lugar de escribir o leer los archivos auxiliares `.clawhub/lock.json` y
  `.clawhub/origin.json` durante la ejecución. El código del entorno de ejecución usa objetos de estado
  de instalaciones registradas en lugar de abstracciones de archivo de bloqueo y origen con forma de archivo. Doctor
  importa los archivos auxiliares heredados desde los espacios de trabajo configurados de los agentes y los elimina
  después de una importación correcta.
- El índice de plugins instalados ahora lee y escribe la fila única SQLite compartida y tipada
  `installed_plugin_index` en lugar de `plugins/installs.json`; el
  archivo JSON heredado es únicamente una entrada de migración de Doctor y se elimina después de importarlo.
- El ayudante de ruta heredado `plugins/installs.json` ahora reside en el código heredado de Doctor.
  Los módulos del índice de plugins del entorno de ejecución solo exponen opciones de persistencia respaldadas por SQLite,
  no una ruta de archivo JSON.
- El estado del centinela de reinicio, la intención de reinicio y el traspaso al supervisor del Gateway ahora usan
  filas SQLite compartidas y tipadas (`gateway_restart_sentinel`,
  `gateway_restart_intent` y `gateway_restart_handoff`) en lugar de blobs genéricos
  opacos. El código de reinicio del entorno de ejecución no tiene ningún contrato de centinela, intención o traspaso
  con forma de archivo.
- La caché de sincronización, los metadatos de almacenamiento, las vinculaciones de hilos, los marcadores de desduplicación
  de entrada, el estado del tiempo de espera de verificación al inicio, las instantáneas criptográficas de IndexedDB del SDK,
  las credenciales y las claves de recuperación de Matrix ahora usan tablas compartidas de estado y blobs
  de plugins SQLite. Las estructuras de rutas del entorno de ejecución ya no exponen una ruta de metadatos `storage-meta.json`;
  ese nombre de archivo es únicamente una entrada de migración heredada. Su plan de importación de JSON heredado
  reside en la superficie de configuración y migración de Doctor del plugin Matrix. Los marcadores de
  desduplicación de entrada usan la desduplicación reclamable del núcleo (espacios de nombres `matrix.inbound-dedupe.*`
  en la base de datos de estado compartida); la migración de estado de Doctor de Matrix importa
  una vez las filas retiradas por raíz `inbound-dedupe` y `inbound-dedupe.json`,
  y después el entorno de ejecución solo lee el almacén de desduplicación reclamable.
- El inicio de Matrix ya no examina, informa ni completa el estado heredado de archivos de Matrix.
  La detección de archivos de Matrix, la creación de instantáneas criptográficas heredadas, el estado de migración
  de restauración de claves de salas, la importación y la eliminación de fuentes son ahora responsabilidad exclusiva de Doctor.
- Se eliminaron los barrels de migración del entorno de ejecución de Matrix. Los ayudantes de detección
  y modificación de estado y criptografía heredados son importados directamente por Doctor de Matrix en lugar de formar
  parte de la superficie de API del entorno de ejecución.
- Los marcadores de reutilización de instantáneas de migración de Matrix ahora residen en el estado de plugin SQLite
  en lugar de `matrix/migration-snapshot.json`; Doctor aún puede reutilizar el mismo
  archivo verificado previo a la migración sin escribir un archivo auxiliar de estado.
- Los cursores del bus y el estado de publicación de perfiles de Nostr ahora usan el estado compartido de plugins SQLite.
  Su plan de importación de JSON heredado reside en la superficie de configuración y migración
  de Doctor del plugin Nostr.
- Los conmutadores de sesión de Active Memory ahora usan el estado compartido de plugins SQLite en lugar de
  `session-toggles.json`; al volver a activar la memoria, se elimina la fila en lugar de
  reescribir un objeto JSON.
- Las propuestas y los contadores de revisión de Skill Workshop ahora usan el estado compartido de plugins SQLite
  en lugar de almacenes `skill-workshop/<workspace>.json` por espacio de trabajo. Cada
  propuesta es una fila independiente bajo `skill-workshop/proposals`, y el contador de
  revisiones es una fila independiente bajo `skill-workshop/reviews`.
- Las ejecuciones de subagentes revisores de Skill Workshop ahora usan el solucionador de transcripciones de sesiones
  del entorno de ejecución en lugar de crear rutas auxiliares de sesión `skill-workshop/<sessionId>.json`.
- Las concesiones de procesos de ACPX ahora usan el estado compartido de plugins SQLite bajo
  `acpx/process-leases` en lugar de un registro de archivo completo `process-leases.json`.
  Cada concesión se almacena en su propia fila, lo que conserva la eliminación de procesos obsoletos al inicio
  sin una ruta de reescritura de JSON durante la ejecución.
- Los scripts contenedores de ACPX y el directorio de inicio aislado de Codex se generan en la
  raíz temporal de OpenClaw. Se vuelven a crear según sea necesario y no son entradas de copia de seguridad ni
  migración.
- La persistencia del registro de ejecuciones de subagentes usa filas compartidas y tipadas `subagent_runs`. La
  antigua ruta `subagents/runs.json` ahora es únicamente una entrada de limpieza de Doctor. Doctor
  la reclama bajo el bloqueo de mantenimiento de estado, registra en SQLite la decisión de descarte
  y la elimina sin importar el estado transitorio de las ejecuciones. No queda ningún lector,
  escritor, caché ni mecanismo de respaldo JSON en el entorno de ejecución; la recuperación entre versiones de ejecuciones
  en curso almacenadas únicamente en archivos no es compatible de forma intencionada en este límite de retirada.
  Las pruebas del entorno de ejecución ya no crean elementos de prueba `runs.json` vacíos o no válidos para demostrar
  el comportamiento del registro; insertan y leen directamente filas SQLite.
- La copia de seguridad prepara el directorio de estado antes de archivarlo, copia los archivos que no son bases de datos,
  crea instantáneas de las bases de datos mediante copia de seguridad en línea más `VACUUM` sin conexión, omite los archivos auxiliares WAL/SHM activos, registra
  los metadatos de las instantáneas en el manifiesto del archivo y registra
  las ejecuciones de copia de seguridad completadas en SQLite junto con el manifiesto del archivo. `openclaw backup
create` valida de forma predeterminada el archivo escrito; `--no-verify` es la
  ruta rápida explícita.
- `openclaw backup restore` valida el archivo antes de extraerlo, reutiliza el
  manifiesto normalizado del verificador y restaura los recursos verificados del manifiesto en sus
  rutas de origen registradas. Requiere `--yes` para realizar escrituras y admite `--dry-run`
  para generar un plan de restauración.
- Se eliminó el antiguo filtro de rutas volátiles de las copias de seguridad. La copia de seguridad ya no necesita una
  lista de exclusión de tar en vivo para archivos JSON/JSONL heredados de sesiones o Cron, porque las instantáneas
  de SQLite se preparan antes de crear el archivo.
- La preparación básica de la configuración y del espacio de trabajo de incorporación ya no crea
  directorios `agents/<agentId>/sessions/`. Solo crea la configuración y el espacio de trabajo;
  las filas de sesiones y transcripciones de SQLite se crean bajo demanda en la
  base de datos de cada agente.
- La reparación de permisos de seguridad ahora se aplica a las bases de datos SQLite
  globales y de cada agente, además de los archivos auxiliares WAL/SHM, en lugar de a `sessions.json` y a los archivos
  JSONL de transcripciones.
- Los nombres de tiempo de ejecución del registro del entorno aislado ahora describen directamente los tipos de registro de SQLite,
  en lugar de mantener la terminología heredada de los registros JSON en el almacén activo.
- `openclaw reset --scope config+creds+sessions` elimina las bases de datos
  `openclaw-agent.sqlite` de cada agente junto con los archivos auxiliares WAL/SHM, no solo los directorios
  heredados `sessions/`.
- Los asistentes de sesiones agregadas del Gateway ahora utilizan nombres orientados a entradas:
  `loadCombinedSessionEntriesForGateway` devuelve `{ databasePath, entries }`.
  La nomenclatura anterior del almacén combinado se ha eliminado de los invocadores en tiempo de ejecución.
- La inicialización de canales MCP de Docker ahora escribe la fila de la sesión principal y los eventos de transcripción
  en la base de datos SQLite de cada agente, en lugar de crear
  `sessions.json` y una transcripción JSONL.
- El hook de memoria de sesión incluido ahora obtiene el contexto de la sesión anterior de
  SQLite mediante `{agentId, sessionId}`. Ya no examina, almacena ni sintetiza
  rutas de transcripciones ni directorios `workspace/sessions`.
- El hook de registro de comandos incluido ahora escribe filas de auditoría de comandos en la tabla
  `command_log_entries` de SQLite compartida, en lugar de añadirlas a
  `logs/commands.log`.
- Las listas de permitidos para el emparejamiento de canales ahora solo exponen asistentes de lectura y escritura respaldados por SQLite en
  tiempo de ejecución. El solucionador de rutas obsoleto del SDK del plugin se mantiene por compatibilidad
  con la migración; los lectores de archivos solo se encuentran en el código de migración de estado de doctor.
- `migration_runs` registra las ejecuciones de migración del estado heredado con su estado,
  marcas de tiempo e informes JSON.
- `migration_sources` registra cada origen de archivo heredado importado con su hash, tamaño,
  número de registros, tabla de destino, id. de ejecución, estado y estado de eliminación del origen.
- `backup_runs` registra las rutas de los archivos de copia de seguridad, el estado y los manifiestos JSON.
- El esquema global no conserva una tabla de registro `agents` sin utilizar. La detección de
  bases de datos de agentes es el registro canónico `agent_databases` hasta que el tiempo de ejecución
  tenga un propietario real de los registros de agentes.
- La configuración generada del catálogo de modelos se almacena en filas tipadas
  `agent_model_catalogs` de SQLite global, identificadas por el directorio del agente. Los invocadores en tiempo de ejecución utilizan
  `ensureOpenClawModelCatalog`; no existe una API de compatibilidad `models.json` en
  el código de tiempo de ejecución. La implementación escribe en SQLite y el registro de PI integrado se
  carga a partir de esa carga útil almacenada sin crear un archivo `models.json`.
- La exportación opcional `memory.qmd.sessions` lee las filas canónicas de transcripciones de
  la base de datos de cada agente y materializa Markdown depurado bajo el directorio principal de QMD
  como un artefacto de entrada explícito de QMD. Por lo tanto, las colecciones de sesiones de QMD y las asignaciones
  de identidad de artefactos siguen formando parte del puente configurado con la herramienta externa;
  no constituyen un segundo almacén canónico de transcripciones.
- Los propios `index.sqlite` de QMD, la configuración YAML de colecciones y las descargas de modelos siguen siendo
  artefactos de la herramienta externa bajo `~/.openclaw/agents/<agentId>/qmd`; no se
  replican en `plugin_blob_entries`. La coordinación de QMD propiedad de OpenClaw
  prioriza la base de datos: los `state_leases` compartidos serializan las incrustaciones globalmente y los
  `state_leases` de cada agente serializan los procesos de escritura de colecciones, actualizaciones e incrustaciones. El tiempo de ejecución no crea
  archivos auxiliares de bloqueo de QMD.
- El plugin opcional `memory-lancedb` ya no crea
  `~/.openclaw/memory/lancedb` como almacén implícito administrado por OpenClaw. Es un
  backend externo de LanceDB y permanece deshabilitado hasta que el operador configure un
  `dbPath` explícito.
- `check:database-first-legacy-stores` rechaza el nuevo código fuente de tiempo de ejecución que vincule
  nombres de almacenes heredados con API de sistema de archivos orientadas a la escritura. También rechaza el código fuente
  de tiempo de ejecución que vuelva a introducir los marcadores retirados del puente de transcripciones
  `transcriptLocator` o `sqlite-transcript://...`. El código de migración, doctor, importación
  y exportación explícita no relacionada con sesiones sigue estando permitido. Los nombres de contratos heredados más amplios,
  como `sessionFile`, `storePath` y las antiguas fachadas de la época de archivos `SessionManager`,
  todavía tienen propietarios actuales y requieren trabajo independiente en las protecciones de migración
  antes de poder convertirse en una comprobación previa obligatoria. La protección ahora también abarca
  los almacenes de tiempo de ejecución `cache/*.json`, los archivos auxiliares genéricos
  `thread-bindings.json`, el estado y los registros de ejecución JSON de Cron, el JSON de estado de configuración,
  los archivos auxiliares de reinicio y bloqueo, la configuración de activación por voz, las aprobaciones de enlaces de plugins,
  el JSON del índice de plugins instalados, el JSONL de auditoría de transferencias de archivos, los registros de actividad
  de Memory Wiki, el antiguo registro de texto incluido `command-logger` y las opciones de diagnóstico JSONL
  de flujos sin procesar de pi-mono. También prohíbe los antiguos nombres de módulos heredados de doctor en el nivel raíz para que
  el código de compatibilidad permanezca bajo `src/commands/doctor/`. Los controladores de depuración de Android
  también utilizan logcat o salida en memoria en lugar de preparar archivos de caché `camera_debug.log` o
  `debug_logs.txt`.

## Forma del esquema de destino

Mantenga los esquemas explícitos. El estado de tiempo de ejecución propiedad del host usa tablas con tipos. El estado opaco
propiedad de plugins usa `plugin_state_entries` / `plugin_blob_entries`; no existe una tabla
genérica del host `kv`.

Base de datos global:

```text
state_leases(scope, lease_key, owner, expires_at, heartbeat_at, payload_json, created_at, updated_at)
exec_approvals_config(config_key, raw_json, socket_path, has_socket_token, default_security, default_ask, default_ask_fallback, auto_allow_skills, agent_count, allowlist_count, updated_at_ms)
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
agent_databases(agent_id, path, schema_version, last_seen_at, size_bytes)
task_runs(...)
task_delivery_state(...)
flow_runs(...)
subagent_runs(run_id, child_session_key, requester_session_key, controller_session_key, created_at, ended_at, cleanup_handled, payload_json)
current_conversation_bindings(binding_key, binding_id, target_agent_id, target_session_id, target_session_key, channel, account_id, conversation_kind, parent_conversation_id, conversation_id, target_kind, status, bound_at, expires_at, metadata_json, updated_at)
plugin_binding_approvals(plugin_root, channel, account_id, plugin_id, plugin_name, approved_at)
tui_last_sessions(scope_key, session_key, updated_at)
plugin_state_entries(plugin_id, namespace, entry_key, value_json, created_at, expires_at)
plugin_blob_entries(plugin_id, namespace, entry_key, metadata_json, blob, created_at, expires_at)
media_blobs(subdir, id, content_type, size_bytes, blob, created_at, updated_at)
skill_uploads(upload_id, kind, slug, force, size_bytes, sha256, actual_sha256, received_bytes, archive_blob, created_at, expires_at, committed, committed_at, idempotency_key_hash)
skill_upload_chunks(upload_id, byte_offset, size_bytes, chunk_blob)
web_push_subscriptions(endpoint_hash, subscription_id, endpoint, p256dh, auth, created_at_ms, updated_at_ms)
web_push_vapid_keys(key_id, public_key, private_key, subject, updated_at_ms)
apns_registrations(node_id, transport, token, relay_handle, send_grant, installation_id, relay_origin, topic, environment, distribution, token_debug_suffix, updated_at_ms)
apns_registration_tombstones(node_id, deleted_at_ms)
node_host_config(config_key, version, node_id, token, display_name, gateway_host, gateway_port, gateway_tls, gateway_tls_fingerprint, gateway_context_path, updated_at_ms)
device_identities(identity_key, device_id, public_key_pem, private_key_pem, created_at_ms, updated_at_ms)
device_auth_tokens(device_id, role, token, scopes_json, updated_at_ms)
macos_port_guardian_records(pid, port, command, mode, timestamp)
workspace_setup_state(workspace_key, workspace_path, version, bootstrap_seeded_at, setup_completed_at, updated_at)
workspace_path_aliases(alias_key, alias_path, workspace_key, workspace_path, updated_at_ms)
workspace_attestations(workspace_key, attested_at_ms, updated_at_ms)
workspace_generated_bootstrap_hashes(workspace_key, filename, sha256)
native_hook_relay_bridges(relay_id, pid, hostname, port, token, expires_at_ms, updated_at_ms)
model_capability_cache(provider_id, model_id, name, input_text, input_image, reasoning, supports_tools, context_window, max_tokens, cost_input, cost_output, cost_cache_read, cost_cache_write, updated_at_ms)
agent_model_catalogs(catalog_key, agent_dir, raw_json, updated_at)
managed_outgoing_image_records(attachment_id, session_key, agent_id, message_id, created_at, updated_at, retention_class, alt, original_media_id, original_media_subdir, original_content_type, original_width, original_height, original_size_bytes, original_filename, record_json, cleanup_pending)
gateway_restart_sentinel(sentinel_key, version, kind, status, ts, session_key, thread_id, delivery_channel, delivery_to, delivery_account_id, message, continuation_json, doctor_hint, stats_json, payload_json, updated_at_ms)
channel_pairing_requests(channel_key, account_id, request_id, code, created_at, last_seen_at, meta_json)
channel_pairing_allow_entries(channel_key, account_id, entry, sort_order, updated_at)
voicewake_triggers(config_key, position, trigger, updated_at_ms)
voicewake_routing_config(config_key, version, default_target_mode, default_target_agent_id, default_target_session_key, updated_at_ms)
voicewake_routing_routes(config_key, position, trigger, target_mode, target_agent_id, target_session_key, updated_at_ms)
update_check_state(state_key, last_checked_at, last_notified_version, last_notified_tag, last_available_version, last_available_tag, auto_install_id, auto_first_seen_version, auto_first_seen_tag, auto_first_seen_at, auto_last_attempt_version, auto_last_attempt_at, auto_last_success_version, auto_last_success_at, updated_at_ms)
config_health_entries(config_path, last_known_good_json, last_promoted_good_json, last_observed_suspicious_signature, updated_at_ms)
sandbox_registry_entries(registry_kind, container_name, session_key, backend_id, runtime_label, image, created_at_ms, last_used_at_ms, config_label_kind, config_hash, cdp_port, no_vnc_port, entry_json, updated_at)
cron_jobs(store_key, job_id, name, description, enabled, delete_after_run, created_at_ms, agent_id, session_key, schedule_kind, schedule_expr, schedule_tz, every_ms, anchor_ms, at, stagger_ms, session_target, wake_mode, payload_kind, payload_message, payload_model, payload_fallbacks_json, payload_thinking, payload_timeout_seconds, payload_allow_unsafe_external_content, payload_external_content_source_json, payload_light_context, payload_tools_allow_json, delivery_mode, delivery_channel, delivery_to, delivery_thread_id, delivery_account_id, delivery_best_effort, failure_delivery_mode, failure_delivery_channel, failure_delivery_to, failure_delivery_account_id, failure_alert_disabled, failure_alert_after, failure_alert_channel, failure_alert_to, failure_alert_cooldown_ms, failure_alert_include_skipped, failure_alert_mode, failure_alert_account_id, next_run_at_ms, running_at_ms, last_run_at_ms, last_run_status, last_error, last_duration_ms, consecutive_errors, consecutive_skipped, schedule_error_count, last_delivery_status, last_delivery_error, last_delivered, last_failure_alert_at_ms, job_json, state_json, runtime_updated_at_ms, schedule_identity, sort_order, updated_at)
delivery_queue_entries(queue_name, id, status, entry_kind, session_key, channel, target, account_id, retry_count, last_attempt_at, last_error, recovery_state, platform_send_started_at, entry_json, enqueued_at, updated_at, failed_at)
commitments(id, agent_id, session_key, channel, account_id, recipient_id, thread_id, sender_id, kind, sensitivity, source, status, reason, suggested_text, dedupe_key, confidence, due_earliest_ms, due_latest_ms, due_timezone, source_message_id, source_run_id, created_at_ms, updated_at_ms, attempts, last_attempt_at_ms, sent_at_ms, dismissed_at_ms, snoozed_until_ms, expired_at_ms, record_json)
migration_runs(id, started_at, finished_at, status, report_json)
migration_sources(source_key, migration_kind, source_path, target_table, source_sha256, source_size_bytes, source_record_count, last_run_id, status, imported_at, removed_source, report_json)
backup_runs(id, created_at, archive_path, status, manifest_json)
```

Base de datos del agente:

```text
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
sessions(session_id, session_key, session_scope, created_at, updated_at, started_at, ended_at, status, chat_type, channel, account_id, primary_conversation_id, model_provider, model, agent_harness_id, parent_session_key, spawned_by, display_name)
conversations(conversation_id, channel, account_id, kind, peer_id, parent_conversation_id, thread_id, native_channel_id, native_direct_user_id, label, metadata_json, created_at, updated_at)
session_conversations(session_id, conversation_id, role, first_seen_at, last_seen_at)
session_routes(session_key, session_id, updated_at)
session_entries(session_id, session_key, entry_json, updated_at)
transcript_events(session_id, seq, event_json, created_at)
transcript_event_identities(session_id, event_id, seq, event_type, has_parent, parent_id, message_idempotency_key, created_at)
transcript_snapshots(session_id, snapshot_id, reason, event_count, created_at, metadata_json)
vfs_entries(namespace, path, kind, content_blob, metadata_json, updated_at)
tool_artifacts(run_id, artifact_id, kind, metadata_json, blob, created_at)
run_artifacts(run_id, path, kind, metadata_json, blob, created_at)
trajectory_runtime_events(session_id, run_id, seq, event_json, created_at)
memory_index_meta(key, value)
memory_index_sources(id, path, source, hash, mtime, size)
memory_index_chunks(id, path, source, start_line, end_line, hash, model, text, embedding, updated_at)
memory_embedding_cache(provider, model, provider_key, hash, embedding, dims, updated_at)
memory_index_state(id, revision)
cache_entries(scope, key, value_json, blob, expires_at, updated_at)
```

`memory_index_sources.id` es la clave primaria entera estable; `(path, source)` sigue siendo única.

Las búsquedas futuras pueden añadir tablas FTS sin cambiar las tablas canónicas de eventos:

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

Los valores grandes deben usar columnas `blob`, no codificación como cadenas JSON. Mantenga
`value_json` para datos estructurados pequeños que deban seguir siendo inspeccionables con herramientas
SQLite sencillas.

`agent_databases` es el registro canónico para esta rama. No añada una tabla
`agents` hasta que exista un propietario real de los registros de agentes; la configuración de agentes permanece en
`openclaw.json`.

## Forma de la migración de Doctor

Doctor debe invocar un paso de migración explícito que pueda incluirse en informes y sea seguro
volver a ejecutar:

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` invoca la implementación de migración del estado después de
la comprobación preliminar ordinaria de la configuración y crea una copia de seguridad verificada antes de la importación. El inicio del
tiempo de ejecución y `openclaw migrate` no deben importar archivos de estado heredados de OpenClaw.

Propiedades de la migración:

- Una pasada de migración detecta todas las fuentes de archivos heredados y genera un plan
  antes de modificar nada.
- Doctor crea un archivo de copia de seguridad verificado previo a la migración antes de importar
  archivos heredados.
- Las importaciones son idempotentes y se identifican por la ruta de origen, la hora de modificación, el tamaño, el hash y la tabla
  de destino.
- Los archivos de origen procesados correctamente se eliminan o archivan después de que se haya confirmado la transacción de la base de datos
  de destino.
- Las importaciones fallidas dejan el origen intacto y registran una advertencia en
  `migration_runs`.
- El código de tiempo de ejecución solo lee SQLite después de que exista la migración.
- No se requiere ninguna ruta de reversión ni de exportación a archivos de tiempo de ejecución.

## Inventario de migración

Mueva estos elementos a la base de datos global:

- Las escrituras en tiempo de ejecución del registro de tareas ahora usan la base de datos compartida; se ha eliminado el importador de archivos auxiliares
  `tasks/runs.sqlite` no publicado. Los guardados de instantáneas realizan una inserción o actualización por id. de tarea
  y eliminan únicamente las filas de tareas/entregas que faltan.
- Las escrituras en tiempo de ejecución de Task Flow ahora usan la base de datos compartida; se ha eliminado el importador de archivos auxiliares
  `tasks/flows/registry.sqlite` no publicado. Los guardados de instantáneas
  realizan una inserción o actualización por id. de flujo y eliminan únicamente las filas de flujo que faltan.
- Las escrituras en tiempo de ejecución del estado de los plugins ahora usan la base de datos compartida; se ha eliminado el importador de archivos auxiliares
  `plugin-state/state.sqlite` no publicado.
- La búsqueda de memoria integrada ya no usa `memory/<agentId>.sqlite` de forma predeterminada; sus
  tablas de índice se encuentran en la base de datos del agente propietario y la habilitación explícita
  del archivo auxiliar `memorySearch.store.path` se ha trasladado a la migración de configuración
  de doctor.
- La reindexación de la memoria integrada restablece únicamente las tablas propiedad de la memoria en la base de datos del agente.
  No debe reemplazar todo el archivo SQLite, porque la misma base de datos contiene
  sesiones, transcripciones, filas de VFS, artefactos y cachés de tiempo de ejecución.
- Registros de contenedores/navegadores de sandbox procedentes de JSON monolítico y fragmentado. Las escrituras en tiempo de ejecución
  ahora usan la base de datos compartida; se mantiene la importación del JSON heredado.
- Las definiciones de trabajos Cron, el estado de la programación y el historial de ejecuciones ahora usan SQLite compartido;
  doctor importa/elimina los archivos heredados `jobs.json`, `jobs-state.json` y
  `cron/runs/*.jsonl`
- Identidad/autenticación de dispositivos, notificaciones push, comprobación de actualizaciones, compromisos, caché de modelos
  de OpenRouter, índice de plugins instalados y enlaces del servidor de aplicaciones
- Los registros de emparejamiento e inicialización de dispositivos/nodos ahora usan tablas SQLite con tipos
- Los suscriptores a notificaciones de emparejamiento de dispositivos y los marcadores de solicitudes entregadas ahora usan la
  tabla compartida de estado de plugins de SQLite en lugar de `device-pair-notify.json`.
- Los registros de llamadas de voz ahora usan la tabla compartida de estado de plugins de SQLite en el
  espacio de nombres `voice-call` / `calls` en lugar de `calls.jsonl`; la CLI del plugin
  sigue y resume el historial de llamadas respaldado por SQLite.
- Las sesiones del Gateway de QQBot, los registros de usuarios conocidos y la caché de citas del índice de referencias ahora usan
  el estado de plugins de SQLite en los espacios de nombres `qqbot` (`gateway-sessions`,
  `known-users`, `ref-index`) en lugar de `session-*.json`, `known-users.json`
  y `ref-index.jsonl`. Esos archivos heredados son cachés y no se migran.
- Las preferencias del selector de modelos de Discord, los hashes de despliegue de comandos y los enlaces de hilos
  ahora usan el estado de plugins de SQLite en los espacios de nombres `discord`
  (`model-picker-preferences`, `command-deploy-hashes`, `thread-bindings`)
  en lugar de `model-picker-preferences.json`, `command-deploy-cache.json` y
  `thread-bindings.json`; la migración de doctor/configuración de Discord importa y
  elimina los archivos heredados.
- Los cursores de puesta al día y los marcadores de deduplicación de entrada de BlueBubbles ahora usan el estado de plugins
  de SQLite en los espacios de nombres `bluebubbles` (`catchup-cursors`, `inbound-dedupe`)
  en lugar de `bluebubbles/catchup/*.json` y
  `bluebubbles/inbound-dedupe/*.json`; la migración de doctor/configuración de BlueBubbles
  importa y elimina los archivos heredados.
- Los desplazamientos de actualizaciones de Telegram, las entradas de la caché de adhesivos, las entradas de la caché de mensajes de cadenas de respuestas,
  las entradas de la caché de mensajes enviados, las entradas de la caché de nombres de temas y los enlaces de hilos
  ahora usan el estado de plugins de SQLite en los espacios de nombres `telegram`
  (`update-offsets`, `sticker-cache`, `message-cache`, `sent-messages`,
  `topic-names`, `thread-bindings`) en lugar de `update-offset-*.json`,
  `sticker-cache.json`, `*.telegram-messages.json`,
  `*.telegram-sent-messages.json`, `*.telegram-topic-names.json` y
  `thread-bindings-*.json`; la migración de doctor/configuración de Telegram importa y
  elimina los archivos heredados.
- Los cursores de puesta al día, las asignaciones de identificadores cortos de respuestas y las filas de deduplicación de ecos enviados de iMessage
  ahora usan el estado de plugins de SQLite en los espacios de nombres `imessage` (`catchup-cursors`,
  `reply-cache`, `sent-echoes`) en lugar de `imessage/catchup/*.json`,
  `imessage/reply-cache.jsonl` y `imessage/sent-echoes.jsonl`; la migración de
  doctor/configuración de iMessage importa y elimina los archivos heredados.
- Las conversaciones, encuestas, tokens SSO y aprendizajes de comentarios de Microsoft Teams ahora
  usan espacios de nombres de estado de plugins de SQLite (`conversations`, `polls`, `sso-tokens`,
  `feedback-learnings`) en lugar de `msteams-conversations.json`,
  `msteams-polls.json`, `msteams-sso-tokens.json` y `*.learnings.json`; la
  migración de doctor/configuración de Microsoft Teams importa y archiva los archivos heredados.
  Las cargas pendientes son una caché de SQLite de corta duración y los archivos de caché JSON antiguos
  no se migran.
- La caché de sincronización, los metadatos de almacenamiento, los enlaces de hilos, los marcadores de deduplicación de entrada,
  el estado de tiempo de espera de la verificación de inicio, las credenciales, las claves de recuperación y las instantáneas
  criptográficas de IndexedDB del SDK de Matrix ahora usan espacios de nombres de blobs/estado de plugins de SQLite en
  `matrix` (`sync-store`, `storage-meta`, `thread-bindings`,
  `matrix.inbound-dedupe.*` mediante la deduplicación reclamable del núcleo,
  `startup-verification`, `credentials`, `recovery-key`, `idb-snapshots`)
  en lugar de `bot-storage.json`, `storage-meta.json`, `thread-bindings.json`,
  `inbound-dedupe.json`, `startup-verification.json`, `credentials.json`,
  `recovery-key.json` y `crypto-idb-snapshot.json`; la migración de doctor/configuración
  de Matrix importa y elimina esos archivos heredados (y las filas SQLite retiradas
  `inbound-dedupe` por raíz) de las raíces de almacenamiento de Matrix con ámbito de cuenta.
- Los cursores del bus y el estado de publicación de perfiles de Nostr ahora usan el estado de plugins de SQLite en
  los espacios de nombres `nostr` (`bus-state`, `profile-state`) en lugar de
  `bus-state-*.json` y `profile-state-*.json`; la migración de doctor/configuración de Nostr
  importa y elimina los archivos heredados.
- Los conmutadores de sesión de Active Memory ahora usan el estado de plugins de SQLite en
  `active-memory/session-toggles` en lugar de `session-toggles.json`.
- Las colas de propuestas y los contadores de revisiones de Skill Workshop ahora usan el estado de plugins de SQLite
  en `skill-workshop/proposals` y `skill-workshop/reviews` en lugar de
  archivos `skill-workshop/<workspace>.json` por espacio de trabajo.
- Las colas de entrega saliente y de entrega de sesiones ahora comparten la tabla global de SQLite
  `delivery_queue_entries` con nombres de cola independientes
  (`outbound-delivery`, `session-delivery`) en lugar de los archivos persistentes
  `delivery-queue/*.json`, `delivery-queue/failed/*.json` y
  `session-delivery-queue/*.json`. El paso de estado heredado de doctor importa
  las filas pendientes y fallidas, elimina los marcadores obsoletos de entregas y borra los archivos
  JSON antiguos después de la importación. Los campos activos de enrutamiento y reintento son columnas con tipos; la
  carga útil JSON se conserva únicamente para reproducción/depuración.
- Los arrendamientos de procesos ACPX ahora usan el estado de plugins de SQLite en `acpx/process-leases`
  en lugar de `process-leases.json`.
- Metadatos de ejecuciones de copias de seguridad y migraciones

Trasladar lo siguiente a las bases de datos de los agentes:

- Raíces de sesiones de agentes y cargas útiles de entradas de sesión con forma de compatibilidad. Completado para
  las escrituras en tiempo de ejecución: los metadatos activos de sesión se pueden consultar en `sessions`, mientras que la
  carga útil completa `SessionEntry` con la forma heredada permanece en `session_entries`.
- Eventos de transcripciones de agentes. Completado para las escrituras en tiempo de ejecución.
- Puntos de control de Compaction e instantáneas de transcripciones. Completado para las escrituras en tiempo de ejecución:
  las copias de transcripciones de los puntos de control son filas de transcripciones de SQLite y los metadatos
  de los puntos de control se registran en `transcript_snapshots`. Los auxiliares de puntos de control del Gateway
  ahora denominan estos valores instantáneas de transcripciones en lugar de archivos de origen.
- Espacios de nombres temporales/de espacios de trabajo del VFS de los agentes. Completado para las escrituras del VFS en tiempo de ejecución.
- Cargas útiles de archivos adjuntos de subagentes. Completado para las escrituras en tiempo de ejecución: son entradas de
  inicialización del VFS de SQLite y nunca archivos persistentes del espacio de trabajo.
- Artefactos de herramientas. Completado para las escrituras en tiempo de ejecución.
- Artefactos de ejecución. Completado para las escrituras en tiempo de ejecución de los trabajadores mediante la tabla por agente
  `run_artifacts`.
- Cachés locales de tiempo de ejecución de los agentes. Completado para las escrituras de caché con ámbito de tiempo de ejecución de los trabajadores mediante
  la tabla por agente `cache_entries`. Las cachés de modelos de todo el Gateway permanecen en la
  base de datos global salvo que pasen a ser específicas de un agente.
- Registros de flujos principales de ACP. Completado para las escrituras en tiempo de ejecución.
- Sesiones del libro mayor de reproducción de ACP. Completado para las escrituras en tiempo de ejecución mediante
  `acp_replay_sessions` y `acp_replay_events`; el archivo heredado `acp/event-ledger.json`
  permanece únicamente como entrada de doctor.
- Metadatos de sesiones de ACP. Completado para las escrituras en tiempo de ejecución mediante `acp_sessions`; los bloques
  heredados `entry.acp` de `sessions.json` son únicamente entradas para la migración de doctor.
- Archivos auxiliares de trayectorias cuando no son archivos de exportación explícitos. Completado para las escrituras en tiempo
  de ejecución: la captura de trayectorias escribe filas `trajectory_runtime_events` en la base de datos del agente
  y replica los artefactos con ámbito de ejecución en SQLite. Los archivos auxiliares heredados son únicamente
  entradas de importación de doctor; la exportación puede materializar nuevas salidas JSONL de paquetes de soporte,
  pero no lee ni migra archivos auxiliares antiguos de trayectorias/transcripciones en tiempo de ejecución.
  La captura de trayectorias en tiempo de ejecución expone el ámbito de SQLite; los auxiliares de rutas JSONL están
  aislados para la compatibilidad con la exportación/depuración y no se vuelven a exportar desde el módulo de tiempo de ejecución.
  Los metadatos de trayectorias del ejecutor integrado registran la identidad
  `{agentId, sessionId, sessionKey}` en lugar de conservar un localizador de transcripción.

Mantener lo siguiente respaldado por archivos por ahora:

- `openclaw.json`
- archivos de credenciales del proveedor o de la CLI
- manifiestos de plugins/paquetes
- espacios de trabajo del usuario y repositorios Git cuando se selecciona el modo de disco
- registros destinados al seguimiento por parte del operador, salvo que se traslade una superficie de registro específica

## Plan de migración

### Fase 0: Fijar el límite

Hacer explícito el límite del estado persistente antes de trasladar más filas:

- Añadir una tabla `migration_runs` a la base de datos global.
  Completado para los informes de ejecución de migraciones de estados heredados.
- Añadir un único servicio de migración de estados propiedad de doctor para la importación de archivos a la base de datos.
  Completado: `openclaw doctor --fix` usa la implementación de migración de estados heredados.
- Hacer que `plan` sea de solo lectura y que `apply` cree una copia de seguridad, importe, verifique y
  después elimine o ponga en cuarentena los archivos antiguos.
  Completado: doctor crea una copia de seguridad verificada previa a la migración, pasa la ruta de la copia de seguridad
  a `migration_runs` y reutiliza las rutas del importador/de eliminación.
- Añadir prohibiciones estáticas para que el nuevo código de tiempo de ejecución no pueda escribir archivos de estado heredados, mientras
  que el código de migración y las pruebas aún puedan crearlos/leerlos.
  Completado para los almacenes heredados migrados actualmente; la protección también analiza las
  pruebas anidadas en busca de contratos prohibidos de localizadores de transcripciones en tiempo de ejecución.

### Fase 1: Completar el plano de control global

Mantener el estado de coordinación compartido en `state/openclaw.sqlite`:

- Agentes y registro de bases de datos de agentes
- Libros mayores de tareas y Task Flow
- Estado de los plugins
- Registro de contenedores/navegadores de sandbox
- Historial de ejecuciones de Cron/planificador
- Emparejamiento, dispositivos, notificaciones push, comprobación de actualizaciones, TUI, cachés de OpenRouter/modelos y otros
  estados pequeños de tiempo de ejecución con ámbito del Gateway
- Metadatos de copias de seguridad y migraciones
- Bytes de archivos adjuntos multimedia del Gateway. Completado para las escrituras en tiempo de ejecución; las rutas directas de archivos
  son materializaciones temporales para mantener la compatibilidad con los emisores de canales y la preparación
  de sandbox. Las listas de permitidos en tiempo de ejecución aceptan rutas de materialización de SQLite, no raíces heredadas
  de contenido multimedia de estado/configuración. Doctor importa los archivos multimedia heredados en
  `media_blobs` y elimina los archivos de origen tras escribir correctamente las filas.
- Sesiones, eventos y blobs de cargas útiles de captura del proxy de depuración. Completado: las capturas se encuentran
  en la base de datos de estado compartida y se abren mediante la inicialización, el esquema,
  WAL y la configuración de tiempo de espera por ocupación de la base de datos de estado compartida. Los bytes de las cargas útiles se comprimen con gzip en
  `capture_blobs.data`; no existe una anulación de la base de datos auxiliar en tiempo de ejecución del proxy de depuración,
  un directorio de blobs ni un destino de esquema/generación de código generado exclusivo para capturas del proxy.
  La migración de doctor/inicio importa las filas publicadas `debug-proxy/capture.sqlite`
  y los blobs de cargas útiles referenciados, incluidas las anulaciones activas del entorno para la base de datos/blobs heredados,
  y después archiva esos orígenes mientras mantiene intactos los certificados de CA.

Esta fase también elimina de esos subsistemas las funciones duplicadas de apertura de archivos auxiliares, los asistentes de permisos, la configuración de WAL, la depuración del sistema de archivos y los escritores de compatibilidad.

### Fase 2: Introducir bases de datos por agente

Crear una base de datos por agente y registrarla desde la base de datos global:

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

La fila global `agent_databases` almacena la ruta, la versión del esquema, la marca de tiempo de la última actividad y metadatos básicos de tamaño e integridad. El código de ejecución solicita al registro la base de datos del agente en lugar de derivar directamente las rutas de los archivos.

La base de datos del agente contiene:

- `sessions` como raíz canónica de la sesión, con `session_entries` como tabla de carga útil con forma de compatibilidad asociada a esa raíz, y
  `session_routes` como búsqueda activa única de `session_key`
- `conversations` y `session_conversations` como identidad normalizada de
  enrutamiento del proveedor asociada a las sesiones
- `transcript_events`
- instantáneas de transcripciones y puntos de control de Compaction. Completado para las escrituras en tiempo de ejecución.
- `vfs_entries`
- `tool_artifacts` y artefactos de ejecución
- filas locales del agente de ejecución/caché. Completado para las cachés con ámbito de trabajador.
- eventos del flujo principal de ACP
- eventos de ejecución de trayectoria cuando no son artefactos explícitos de exportación

### Fase 3: Sustituir las API del almacén de sesiones

Completado para el tiempo de ejecución. La superficie del almacén de sesiones con forma de archivo no es un contrato de ejecución activo:

- El tiempo de ejecución ya no llama a `loadSessionStore(storePath)` ni trata `storePath` como
  identidad de sesión.
- Las operaciones de filas en tiempo de ejecución son `getSessionEntry`, `upsertSessionEntry`,
  `patchSessionEntry`, `deleteSessionEntry` y `listSessionEntries`.
- Los asistentes de reescritura de todo el almacén, los escritores de archivos, las pruebas de colas, la depuración de alias y
  los parámetros de eliminación de claves heredadas se han eliminado del tiempo de ejecución.
- Las exportaciones de compatibilidad obsoletas del paquete raíz delegan en el importador
  `sessions.json`, exclusivo de doctor, hasta 2026-10-12; las lecturas de compatibilidad del SDK de Plugin
  siguen proyectando filas SQLite canónicas.
- El análisis de `sessions.json` permanece únicamente en el código de migración/importación de doctor y
  en las pruebas de doctor.
- Las lecturas de reserva del ciclo de vida en tiempo de ejecución leen las cabeceras de las transcripciones de SQLite, no las primeras
  líneas de JSONL.

Seguir eliminando todo lo que reintroduzca parámetros de bloqueo de archivos, vocabulario de depuración/truncamiento como mantenimiento de archivos, identidad basada en la ruta del almacén o pruebas cuya única aserción sea la persistencia de JSON.

### Fase 4: Trasladar transcripciones, flujos ACP, trayectorias y VFS

Hacer que todos los flujos de datos de los agentes sean nativos de la base de datos:

- Las escrituras de anexado de transcripciones pasan por una única transacción SQLite que garantiza la
  cabecera de la sesión, comprueba la idempotencia del mensaje, selecciona el extremo principal, inserta
  en `transcript_events` y registra metadatos de identidad consultables en
  `transcript_event_identities`. Completado para los anexados directos de mensajes de transcripción y
  los anexados persistidos normales de `TranscriptSessionManager`; las operaciones explícitas de rama
  conservan su selección explícita del elemento principal y siguen escribiendo filas SQLite
  sin derivar ningún localizador de archivos.
- Los registros del flujo principal de ACP pasan a ser filas, no archivos `.acp-stream.jsonl`. Completado.
- La configuración de creación de ACP ya no persiste rutas de transcripciones JSONL. Completado.
- La captura de trayectorias en tiempo de ejecución escribe directamente filas de eventos/artefactos. El comando explícito
  de soporte/exportación todavía puede producir artefactos JSONL de paquetes de soporte como
  formato de exportación, pero la exportación de sesiones no vuelve a crear JSONL de sesiones. Completado.
- Los espacios de trabajo en disco permanecen en el disco cuando se configura el modo de disco.
- El espacio temporal de VFS y el modo experimental de espacio de trabajo exclusivo de VFS utilizan la base de datos del agente.

La migración importa una sola vez los archivos JSONL antiguos, registra recuentos/hashes en
`migration_runs` y elimina los archivos importados después de las comprobaciones de integridad.

### Fase 5: Copia de seguridad, restauración, Vacuum y verificación

Las copias de seguridad siguen siendo un único archivo comprimido:

- Crear un punto de control de todas las bases de datos globales y de agentes.
- Crear una instantánea de cada base de datos mediante la copia de seguridad en línea de SQLite, seguida de `VACUUM` sin conexión.
- Archivar las instantáneas compactas de las bases de datos, la configuración, las credenciales externas y las exportaciones
  solicitadas de espacios de trabajo.
- Omitir los archivos activos sin procesar `*.sqlite-wal` y `*.sqlite-shm`.
- Verificar abriendo cada instantánea de base de datos y ejecutando `PRAGMA integrity_check`.
  `openclaw backup create` realiza esta verificación del archivo comprimido de forma predeterminada;
  `--no-verify` omite únicamente la pasada posterior a la escritura del archivo comprimido, no la comprobación de integridad
  durante la creación de la instantánea.
- La restauración copia las instantáneas de vuelta a sus rutas de destino. Las bases de datos globales restauradas utilizan
  la versión `1`; las bases de datos por agente restauradas utilizan la versión `2`, y las instantáneas de la versión `1`
  se actualizan atómicamente al abrirse.

### Fase 6: Ejecución de trabajadores

Mantener experimental el modo de trabajador mientras se implementa la división de las bases de datos:

- Los trabajadores reciben el identificador del agente, el identificador de ejecución, el modo del sistema de archivos y la identidad del registro de bases de datos.
- Cada trabajador abre su propia conexión SQLite.
- El proceso principal conserva la autoridad sobre la entrega del canal, las aprobaciones, la configuración y la cancelación.
- Comenzar con un trabajador por ejecución activa; añadir agrupación únicamente después de que el ciclo de vida y la propiedad de las
  conexiones a la base de datos sean estables.

### Fase 7: Eliminar el mundo anterior

Completado para la gestión de sesiones en tiempo de ejecución. El mundo anterior solo se permite como entrada explícita
de doctor o salida de soporte/exportación:

- No se realizan escrituras en tiempo de ejecución de `sessions.json`, JSONL de transcripciones, JSON del registro del entorno aislado, SQLite
  auxiliar de tareas ni SQLite auxiliar del estado de plugins.
- No se realiza depuración de archivos JSON/de sesión, truncamiento de transcripciones en archivos, bloqueos de archivos de sesión
  ni pruebas de sesión con forma de bloqueo.
- No hay exportaciones de compatibilidad en tiempo de ejecución cuyo propósito sea mantener actualizados los archivos de sesión
  antiguos.
- Las exportaciones explícitas de soporte siguen siendo formatos de archivo/materialización solicitados por el usuario
  y no deben devolver nombres de archivos a la identidad en tiempo de ejecución.

## Copia de seguridad y restauración

Las copias de seguridad deben ser un único archivo comprimido, pero la captura de las bases de datos debe ser
nativa de SQLite:

1. Mantener acotadas las transacciones de escritura para que la copia de seguridad en línea pueda avanzar.
2. Verificar todas las bases de datos globales y de agentes activas antes de la captura.
3. Capturar cada base de datos mediante la copia de seguridad en línea de SQLite en un directorio temporal de
   copia de seguridad; después, cerrar la conexión activa y aplicar `VACUUM` a la copia privada.
   Los esquemas de plugins que requieran capacidades SQLite definidas por su propietario fallan de forma cerrada
   hasta que el propietario proporcione un contrato seguro de instantáneas.
4. Archivar las instantáneas de las bases de datos, el archivo de configuración, el directorio de credenciales, los espacios de trabajo
   seleccionados y un manifiesto.
5. Verificar la estructura de archivo de cada instantánea SQLite; después, abrir las bases de datos canónicas de OpenClaw
   y ejecutar `PRAGMA integrity_check` junto con la validación de roles. Los esquemas dedicados de
   plugins permanecen opacos a menos que su propietario proporcione un verificador.
   `openclaw backup create` realiza esta operación de forma predeterminada; `--no-verify` solo sirve para
   omitir deliberadamente la pasada posterior a la escritura del archivo comprimido.

No utilizar copias directas de los archivos activos `*.sqlite`, `*.sqlite-wal` y `*.sqlite-shm` como
formato principal de copia de seguridad. El manifiesto del archivo comprimido debe registrar el rol de la base de datos,
el identificador del agente, la versión del esquema, la ruta de origen, la ruta de la instantánea, el tamaño en bytes y el estado de
integridad.

La restauración debe reconstruir la base de datos global y los archivos de bases de datos de agentes a partir de las
instantáneas del archivo comprimido. El esquema global permanece en la versión `1`; las instantáneas por agente de la versión `1`
reciben la actualización acotada en tiempo de ejecución a la versión `2`. Doctor sigue siendo
el único responsable de la importación de archivos a bases de datos. El comando de restauración valida primero el
archivo comprimido y después sustituye cada recurso del manifiesto por la carga útil extraída y verificada.

## Plan de refactorización del tiempo de ejecución

1. Añadir API del registro de bases de datos.
   - Resolver las rutas de la base de datos global y de las bases de datos por agente.
   - Mantener el esquema global en `user_version = 1`. Las bases de datos por agente utilizan la versión `2`
     con una migración atómica desde la estructura de origen de memoria de la versión publicada `1`.
   - Añadir asistentes de cierre, punto de control e integridad utilizados por las pruebas, la copia de seguridad y doctor.

2. Consolidar los almacenes SQLite auxiliares.
   - Trasladar las tablas de estado de plugins a la base de datos global. Completado para las escrituras en tiempo de
     ejecución; se ha eliminado el importador del almacén auxiliar heredado no publicado.
   - Trasladar las tablas del registro de tareas a la base de datos global. Completado para las escrituras en tiempo de
     ejecución; se ha eliminado el importador del almacén auxiliar heredado no publicado.
   - Trasladar las tablas de TaskFlow a la base de datos global. Completado para las escrituras en tiempo de ejecución;
     se ha eliminado el importador del almacén auxiliar heredado no publicado.
   - Trasladar las tablas integradas de búsqueda en memoria a cada base de datos de agente. Completado; doctor elimina ahora
     el valor personalizado explícito `memorySearch.store.path` mediante la migración de configuración.
     La reindexación completa se ejecuta en el mismo lugar únicamente sobre las tablas de memoria; se han eliminado la antigua ruta de
     intercambio del archivo completo y el asistente de intercambio del índice auxiliar.
   - Eliminar las funciones duplicadas de apertura de bases de datos, la configuración de WAL, los asistentes de permisos y las
     rutas de cierre de esos subsistemas.

3. Trasladar las tablas propiedad de los agentes a bases de datos por agente.
   - Crear la base de datos del agente bajo demanda mediante el registro de la base de datos global. Completado.
   - Trasladar las entradas de sesiones en tiempo de ejecución, los eventos de transcripciones, las filas de VFS y los artefactos de
     herramientas a las bases de datos de los agentes. Completado.
   - No migrar las entradas de sesión, los eventos de transcripción, las filas de VFS ni los artefactos de herramientas
     de la base de datos compartida local de la rama; esa disposición nunca se publicó. Conservar únicamente la importación
     heredada de archivos a bases de datos en doctor.

4. Sustituir las API del almacén de sesiones.
   - Eliminar `storePath` como identidad en tiempo de ejecución. Completado para el tiempo de ejecución y protegido
     mediante `check:database-first-legacy-stores`: los metadatos de sesión, las actualizaciones de rutas,
     la persistencia de comandos, la limpieza de sesiones de la CLI, las vistas previas de razonamiento de Feishu,
     la persistencia del estado de las transcripciones, la profundidad de los subagentes, las anulaciones de sesión
     del perfil de autenticación, la lógica de bifurcación principal y la inspección de QA Lab ahora resuelven la
     base de datos a partir de claves canónicas de agente/sesión.
     Las respuestas de la lista de sesiones de Gateway/TUI/UI/macOS ahora exponen `databasePath`
     en lugar del valor heredado `path`; las superficies de depuración de macOS muestran la base de datos por agente
     como estado de solo lectura en lugar de escribir la configuración `session.store`.
     `/status`, la exportación de trayectorias impulsada por el chat y los proxies de dependencias de la CLI ya
     no propagan rutas de almacenes heredados; la reserva del uso de transcripciones lee
     SQLite mediante la identidad del agente/sesión. Las pruebas del tiempo de ejecución y del puente ya no exponen
     `storePath`; las entradas de doctor/migración son propietarias de ese nombre de campo heredado.
     La carga combinada de sesiones de Gateway ya no tiene una rama especial en tiempo de ejecución para
     los valores no basados en plantillas de `session.store`; agrega las filas SQLite por agente.
     Se eliminaron la vía heredada de doctor para bloqueos de sesión y su asistente de limpieza `.jsonl.lock`;
     SQLite es ahora el límite de concurrencia de las sesiones.
     Los puntos de llamada de ejecución críticos utilizan nombres de asistentes orientados a filas, como
     `resolveSessionRowEntry`; el alias de compatibilidad antiguo `resolveSessionStoreEntry`
     se ha eliminado de las exportaciones del tiempo de ejecución y del SDK de Plugin.

- Usar operaciones de fila de `{ agentId, sessionKey }`.
  Hecho: `getSessionEntry`, `upsertSessionEntry`, `deleteSessionEntry`,
  `patchSessionEntry` y `listSessionEntries` son API centradas en SQLite que no
  requieren una ruta al almacén de sesiones. El resumen de estado, el estado
  local del agente, la salud y el comando de listado `openclaw sessions` ahora
  leen directamente las filas de cada agente y muestran las rutas de las bases
  de datos SQLite de cada agente en lugar de rutas de `sessions.json`.
- Reemplazar la eliminación/inserción del almacén completo por `upsertSessionEntry`,
  `deleteSessionEntry`, `listSessionEntries` y consultas SQL de limpieza.
  Hecho para el entorno de ejecución: las rutas críticas ahora usan API de filas
  y parches de filas con reintentos ante conflictos; los auxiliares restantes
  de importación/reemplazo del almacén completo se limitan al código de
  importación de migraciones y a las pruebas del backend SQLite.
  - Eliminar `store-writer.ts` y las pruebas de la cola de escritura. Hecho.
  - Eliminar del entorno de ejecución la depuración de claves heredadas y los
    parámetros de eliminación de alias de las inserciones con actualización y
    los parches de filas de sesión. Hecho.

5. Eliminar el comportamiento del registro JSON del entorno de ejecución.
   - Hacer que las lecturas y escrituras del registro del sandbox usen solo SQLite. Hecho.
   - Importar JSON monolítico y fragmentado solo desde el paso de migración. Hecho.
   - Eliminar los bloqueos del registro fragmentado y las escrituras JSON. Hecho.

- Mantener una única tabla de registro tipada en lugar de almacenar las filas
  del registro como JSON opaco genérico si la estructura sigue siendo estado
  operativo de una ruta crítica. Hecho.

6. Eliminar la mutación de sesiones basada en bloqueos de archivos.
   - Hecho para la creación de bloqueos y las API de bloqueo del entorno de ejecución.
   - Se eliminó la vía independiente de limpieza heredada `.jsonl.lock` de doctor.
   - La integridad del estado ya no tiene una ruta separada para depurar archivos
     de transcripción huérfanos; la migración de doctor importa/elimina las
     fuentes JSONL heredadas en un solo lugar.
   - La coordinación de la instancia única del Gateway usa filas SQLite tipadas
     `state_leases` bajo `gateway_locks` y ya no expone un punto de integración
     de directorio de bloqueo de archivos.
   - La persistencia genérica de deduplicación del SDK de plugins ya no usa
     bloqueos de archivos ni archivos JSON; escribe filas compartidas de estado
     de plugins en SQLite. Hecho.
   - La coordinación de QMD usa un arrendamiento SQLite compartido para las
     incrustaciones y un arrendamiento SQLite por agente para cada proceso de
     escritura de colecciones, actualizaciones e incrustaciones. El entorno de
     ejecución ya no crea `qmd/embed.lock.lock` ni `agents/<agentId>/qmd-write.lock.lock`;
     doctor elimina únicamente los archivos auxiliares retirados que están
     definitivamente obsoletos. Hecho.

7. Hacer que los workers reconozcan la base de datos.
   - Los workers abren sus propias conexiones SQLite.
   - El proceso padre posee la entrega, las devoluciones de llamada de canales y la configuración.
   - El worker recibe el identificador del agente, el identificador de la
     ejecución, el modo del sistema de archivos y la identidad del registro de
     la base de datos, no identificadores activos.
   - `vfs-only` sigue siendo experimental y usa la base de datos del agente
     como raíz de almacenamiento.
   - Mantener primero un worker por ejecución activa. La agrupación puede esperar
     hasta que la duración de las conexiones a la base de datos y el
     comportamiento de cancelación sean rutinarios.

8. Integración de copias de seguridad.
   - Hacer que las copias de seguridad creen instantáneas de las bases de datos
     globales, de agentes y de plugins mediante una copia de seguridad en línea
     seguida de `VACUUM` sin conexión. Hecho para los archivos
     `*.sqlite` detectados bajo el recurso de estado; los esquemas de
     plugins que requieren capacidades no disponibles del propietario fallan de
     forma segura.
   - Añadir verificación de las copias de seguridad para la integridad canónica de
     SQLite y la identidad del esquema, además de validación genérica de la
     estructura de archivos para instantáneas específicas de plugins. Hecho para
     la creación de copias de seguridad y la verificación predeterminada de
     archivos.
   - Registrar los metadatos de las ejecuciones de copias de seguridad en SQLite.
     Hecho mediante la tabla compartida `backup_runs`, con la ruta del
     archivo, el estado y el JSON del manifiesto.
   - Añadir la restauración desde instantáneas de archivos verificadas. Hecho:
     `openclaw backup
restore` valida antes de la extracción, usa el manifiesto normalizado
     del verificador, admite `--dry-run` y requiere `--yes` antes de
     reemplazar las rutas de origen registradas.
   - Incluir la exportación de VFS/espacio de trabajo solo cuando se solicite; no
     exportar los componentes internos de las sesiones como JSON o JSONL.

9. Eliminar pruebas y código obsoletos. Hecho para las superficies conocidas de sesiones del entorno de ejecución.

- Eliminar las pruebas que comprueban la creación durante la ejecución de
  `sessions.json` o de archivos JSONL de transcripciones. Hecho para el
  almacén central de sesiones, el chat, los eventos de transcripción del
  Gateway, la vista previa, el ciclo de vida, las actualizaciones de entradas de
  sesión de comandos, el restablecimiento/seguimiento de respuestas automáticas
  y los accesorios de Dreaming de memory-core, el enrutamiento de destinos de
  aprobación, la reparación de transcripciones de sesiones, la reparación de
  permisos de seguridad, la exportación de trayectorias y la exportación de
  sesiones.
  Las pruebas de transcripciones de Active Memory ahora comprueban los ámbitos
  SQLite y que no se creen archivos JSONL temporales ni persistentes.
  Se eliminó la antigua regresión de depuración de transcripciones de Heartbeat
  porque el entorno de ejecución ya no trunca transcripciones JSONL.
  Las pruebas de la herramienta de listado de sesiones de agentes ya no modelan
  rutas heredadas `sessions.json` como la estructura de respuesta del Gateway;
  las pruebas de la aplicación, la interfaz de usuario y macOS usan
  `databasePath`.
  Las pruebas de uso de transcripciones de `/status` ahora insertan
  directamente filas de transcripciones SQLite en lugar de escribir archivos
  JSONL.
  Las pruebas del ciclo de vida de sesiones del Gateway ahora usan directamente
  auxiliares para insertar datos iniciales de transcripciones SQLite; la
  antigua estructura de accesorio de archivo de sesión de una sola línea ha
  desaparecido de la cobertura de restablecimiento y eliminación.
  `sessions.delete` ya no devuelve un campo `archived: []` de la época de los
  archivos; la eliminación solo informa del resultado de la mutación de filas.
  La antigua opción `deleteTranscript` también ha desaparecido: al eliminar una
  sesión se elimina la raíz canónica `sessions` y se permite que SQLite
  elimine en cascada las filas de transcripciones, instantáneas y trayectorias
  propiedad de la sesión, por lo que ningún llamador puede dejar transcripciones
  huérfanas ni olvidar una rama de limpieza.
  Las pruebas de captura de trayectorias del motor de contexto ahora leen filas
  `trajectory_runtime_events` de una base de datos aislada del agente en lugar de leer
  `session.trajectory.jsonl`.
  Los scripts de datos iniciales de canales MCP de Docker ahora insertan
  directamente filas SQLite. Las escrituras directas de `sessions.json` se
  limitan a los accesorios de doctor.
  Las pruebas E2E de Tool Search Gateway leen las pruebas de llamadas a
  herramientas de las filas de transcripciones SQLite en lugar de examinar
  archivos `agents/<agentId>/sessions/*.jsonl`.
  Los eventos del host de memory-core y las filas temporales del corpus de
  sesiones ahora residen en el estado compartido de plugins en SQLite;
  `events.jsonl` y `session-corpus/*.txt` son únicamente entradas de migración
  heredadas de doctor. Las filas activas usan rutas virtuales
  `memory/session-ingestion/`, no `.dreams/session-corpus`. Se eliminaron el antiguo módulo de
  reparación de Dreaming de memory-core y sus pruebas de CLI/Gateway porque el
  entorno de ejecución ya no posee la reparación del archivo de ese corpus.
  Las pruebas del puente/artefacto público de memory-core ya no muestran
  `.dreams/events.jsonl`; usan el nombre del artefacto JSON virtual respaldado por
  SQLite.
  La documentación pública de pruebas de SDK/Codex ahora indica estado de sesión
  SQLite en lugar de archivos de sesión, y el ejemplo de turno de canal ya no
  expone un argumento `storePath`.
  El estado de sincronización de Matrix ahora usa directamente el almacén de
  estado de plugins de SQLite. Los contratos activos del cliente/entorno de
  ejecución pasan una raíz de almacenamiento de cuenta, no una ruta
  `bot-storage.json`, y doctor importa `bot-storage.json` heredado en SQLite antes
  de eliminar el origen. Los escenarios destructivos y de reinicio de Matrix en
  QA Lab ahora mutan directamente la fila de sincronización SQLite en lugar de
  crear o eliminar archivos `bot-storage.json` falsos, y el sustrato E2EE pasa
  una raíz del almacén de sincronización en lugar de una ruta
  `sync-store.json` falsa.
  La selección de la raíz de almacenamiento de Matrix ya no puntúa las raíces
  según archivos JSON heredados de sincronización/hilos; usa metadatos duraderos
  de la raíz junto con el estado criptográfico real.
  El conjunto de pruebas del backend SQLite de sesiones del entorno de ejecución
  ya no fabrica un `sessions.json`; los accesorios de fuentes heredadas ahora
  residen en las pruebas de doctor que los importan.
  Las pruebas de sesiones del Gateway ya no exponen un auxiliar
  `createSessionStoreDir` ni una configuración de ruta temporal del almacén de sesiones
  sin usar; los directorios de accesorios son explícitos y la configuración
  directa de filas usa la nomenclatura de filas de sesión SQLite.
  La cobertura del analizador del almacén de sesiones JSON5 exclusivo de doctor
  se trasladó de las pruebas de infraestructura a las pruebas de migración de
  doctor, de modo que los conjuntos de pruebas del entorno de ejecución ya no
  poseen el análisis de archivos de sesión heredados.
  Las pruebas de SSO/cargas pendientes del entorno de ejecución de Microsoft
  Teams ya no incluyen accesorios ni analizadores de archivos auxiliares JSON;
  el análisis de tokens SSO heredados reside únicamente en el módulo de
  migración del plugin. Las pruebas de Telegram ya no insertan rutas falsas del
  almacén `/tmp/*.json`; restablecen directamente la caché de mensajes
  respaldada por SQLite. El auxiliar genérico del estado de pruebas de OpenClaw
  ya no expone un escritor heredado `auth-profiles.json`; las pruebas de migración
  de autenticación de doctor poseen ese accesorio localmente.
  Las pruebas del entorno de ejecución para punteros de última sesión de TUI,
  aprobaciones de ejecución, conmutadores de Active Memory,
  deduplicación/verificación de inicio de Matrix, sincronización de fuentes de
  Memory Wiki, vinculaciones de conversaciones actuales, autenticación de
  incorporación e importaciones de secretos de Hermes ya no fabrican antiguos
  archivos auxiliares ni comprueban que no existan nombres de archivo antiguos.
  Demuestran el comportamiento mediante filas SQLite y API públicas del
  almacén; las pruebas de doctor/migración son el único lugar donde deben
  aparecer nombres de archivos de origen heredados.
  Las pruebas del entorno de ejecución para emparejamiento de dispositivos/Node,
  allowFrom de canales, intenciones de reinicio, traspaso de reinicio, entradas
  de la cola de entrega de sesiones, salud de la configuración, cachés de
  iMessage, trabajos de Cron, encabezados de transcripciones de PI, registros de
  subagentes y archivos adjuntos de imágenes gestionadas tampoco crean ya
  archivos JSON/JSONL retirados solo para demostrar que se ignoran o no existen.
  La recuperación por desbordamiento de PI ya no tiene un mecanismo alternativo
  de reescritura/truncamiento de SessionManager: el truncamiento de resultados
  de herramientas y las reescrituras de transcripciones del motor de contexto
  mutan las filas de transcripciones SQLite y luego actualizan desde la base de
  datos el estado activo del prompt.
  Las adiciones persistentes de mensajes de SessionManager se delegan al
  auxiliar de adición atómica de transcripciones SQLite para la selección del
  padre y la idempotencia. Las adiciones normales de metadatos/entradas
  personalizadas también seleccionan el padre actual dentro de SQLite, de modo
  que las instancias obsoletas del gestor no reactivan las condiciones de
  carrera anteriores a SQLite en la cadena de padres.
  La limpieza del final sintético de PI para las comprobaciones previas durante
  un turno y `sessions_yield` ahora recorta directamente el estado de las
  transcripciones SQLite; se eliminaron el antiguo puente de eliminación del
  final de SessionManager y sus pruebas.
  La captura de puntos de control de Compaction también crea instantáneas solo
  desde SQLite; los llamadores ya no pasan un SessionManager activo como fuente
  alternativa de transcripciones.
- Mantener las pruebas que insertan archivos heredados únicamente para la migración.
- La comprobación mediante archivos JSON se ha reemplazado por comprobaciones
  mediante filas SQL para las superficies activas del entorno de ejecución.

- Añadir prohibiciones estáticas para las escrituras del entorno de ejecución
  en rutas JSON heredadas de sesiones/cachés. Hecho para la protección del
  repositorio.

10. Hacer auditable el informe de migración.
    - Registrar las ejecuciones de migración en SQLite con marcas de tiempo de
      inicio/finalización, rutas de origen, hashes de origen, recuentos,
      advertencias y ruta de la copia de seguridad.
      Hecho: las ejecuciones de migración del estado heredado ahora conservan un
      informe `migration_runs` con el inventario de rutas/tablas de origen, el
      SHA-256 del archivo de origen, los tamaños, los recuentos de registros, las
      advertencias y la ruta de la copia de seguridad.
      Hecho: las ejecuciones de migración del estado heredado también conservan
      filas `migration_sources` para la auditoría de cada origen y las futuras
      decisiones de omisión/relleno.
    - Hacer que la aplicación sea idempotente. Volver a ejecutarla después de una
      importación parcial debe omitir un origen ya importado o combinarlo
      mediante una clave estable.
      Hecho: los índices de sesiones, las transcripciones, las colas de entrega,
      el estado de plugins, los registros de tareas y las filas SQLite globales
      propiedad de agentes se importan mediante claves estables o semántica de
      inserción con actualización/reemplazo, de modo que las nuevas ejecuciones
      se combinan sin duplicar filas duraderas.
    - Las importaciones fallidas deben conservar el archivo de origen original en su lugar.
      Hecho: las importaciones fallidas de transcripciones ahora dejan el origen
      JSONL original en la ruta detectada, y `migration_sources` registra el origen
      como `warning` con `removed_source=0` para la siguiente ejecución
      de doctor.

## Reglas de rendimiento

- Una conexión por hilo/proceso está bien; no se deben compartir identificadores entre
  trabajadores.
- Se deben usar WAL, `foreign_keys=ON`, un tiempo de espera por ocupación de 5s y transacciones de escritura `BEGIN IMMEDIATE`
  breves. No se deben superponer reintentos de bloqueo síncronos sobre la
  única espera por ocupación de SQLite.
- Los auxiliares de transacciones de escritura deben seguir siendo síncronos salvo que/hasta que una API de transacciones asíncrona
  añada una semántica explícita de exclusión mutua/contrapresión.
- Las escrituras de entrega al proceso padre deben ser pequeñas y transaccionales.
- Deben evitarse las reescrituras de todo el almacén; se deben usar upsert/delete por fila.
- Se deben añadir índices para las rutas de listado por agente, listado por sesión, fecha de actualización, id de ejecución y
  caducidad antes de trasladar código crítico.
- Los artefactos grandes, los archivos multimedia y los vectores deben almacenarse como BLOB o filas BLOB fragmentadas, no
  como base64 ni JSON de matrices numéricas.
- Las entradas opacas del estado de plugins deben mantenerse pequeñas y delimitadas.
- Se debe añadir limpieza SQL para TTL/caducidad en lugar de depuración del sistema de archivos.
  Hecho para los almacenes de ejecución pertenecientes a la base de datos: los archivos multimedia, el estado de plugins, los blobs de plugins,
  la deduplicación persistente y la caché de agentes caducan mediante filas de SQLite. La limpieza restante
  del sistema de archivos se limita a materializaciones temporales o comandos explícitos
  de eliminación.

## Prohibiciones estáticas

Se debe añadir una comprobación del repositorio que rechace nuevas escrituras en tiempo de ejecución en rutas de estado heredadas:

- `sessions.json`
- `*.trajectory.jsonl` excepto las salidas materializadas de paquetes de soporte
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` archivos de caché en tiempo de ejecución
- `agents/<agentId>/agent/auth.json`
- `agents/<agentId>/agent/models.json`
- `credentials/oauth.json`
- `github-copilot.token.json`
- `openrouter-models.json`
- `auth-profiles.json`
- `auth-state.json`
- `exec-approvals.json`
- `openclaw-workspace-state.json`
- `workspace-state.json`
- `workspace-attestations/*.attested`
- `<workspace>.attested` hermano
- `credentials*.json` y `recovery-key.json` de Matrix
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  (retirado en 2026.7: el almacén de ejecución es `device_pairing_*` /
  `device_bootstrap_tokens` en la base de datos de estado compartida; los registros emparejados se importan al
  iniciar el Gateway y las filas transitorias pendientes/de arranque se descartan)
- `nodes/pending.json` / `nodes/paired.json` (retirado en 2026.7: se integra en los registros de dispositivos emparejados al iniciar el Gateway)
- `identity/device.json`
- `identity/device-auth.json` (retirado; importación exclusiva de Doctor en `device_auth_tokens`)
- `push/web-push-subscriptions.json` (retirado; importación exclusiva de Doctor en `web_push_subscriptions`)
- `push/vapid-keys.json` (retirado; importación exclusiva de Doctor en `web_push_vapid_keys`)
- `push/apns-registrations.json` (retirado; importación exclusiva de Doctor en `apns_registrations`)
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- `.dreams/events.jsonl` de Memory-core
- `.dreams/session-corpus/` de Memory-core
- `.dreams/daily-ingestion.json` de Memory-core
- `.dreams/session-ingestion.json` de Memory-core
- `.dreams/short-term-recall.json` de Memory-core
- `.dreams/phase-signals.json` de Memory-core
- `.dreams/short-term-promotion.lock` de Memory-core
- `skill-workshop/<workspace>.json` de Skill Workshop
- `skill-workshop/skill-workshop-review-*.json` de Skill Workshop
- `bus-state-*.json` de Nostr
- `profile-state-*.json` de Nostr
- `calls.jsonl`
- `known-users.json`
- `ref-index.jsonl`
- `session-*.json` de QQBot
- `bluebubbles/catchup/*.json` de BlueBubbles
- `bluebubbles/inbound-dedupe/*.json` de BlueBubbles
- `update-offset-*.json` de Telegram
- `sticker-cache.json` de Telegram
- `*.telegram-messages.json` de Telegram
- `*.telegram-sent-messages.json` de Telegram
- `*.telegram-topic-names.json` de Telegram
- `thread-bindings-*.json` de Telegram
- `catchup/*.json` de iMessage
- `reply-cache.jsonl` de iMessage
- `sent-echoes.jsonl` de iMessage
- `msteams-conversations.json` de Microsoft Teams
- `msteams-polls.json` de Microsoft Teams
- `msteams-sso-tokens.json` de Microsoft Teams
- `*.learnings.json` de Microsoft Teams
- `bot-storage.json` de Matrix
- `sync-store.json` de Matrix
- `thread-bindings.json` de Matrix
- `inbound-dedupe.json` de Matrix
- `startup-verification.json` de Matrix
- `storage-meta.json` de Matrix
- `crypto-idb-snapshot.json` de Matrix
- `model-picker-preferences.json` de Discord
- `command-deploy-cache.json` de Discord
- archivos JSON de fragmentos del registro de entornos aislados
- `plugin-state/state.sqlite`
- archivos auxiliares de ejecución `openclaw-state.sqlite` ad hoc
- `tasks/runs.sqlite`
- `tasks/flows/registry.sqlite`
- `bindings/current-conversations.json`
- `restart-sentinel.json`
- `gateway-restart-intent.json`
- `gateway-supervisor-restart-handoff.json`
- `gateway.<hash>.lock`
- `qmd/embed.lock.lock`
- `agents/<agentId>/qmd-write.lock.lock`
- `commands.log`
- `config-health.json`
- `port-guard.json`
- `settings/voicewake.json`
- `settings/voicewake-routing.json`
- `plugin-binding-approvals.json`
- `plugins/installs.json`
- `audit/file-transfer.jsonl`
- `audit/crestodian.jsonl`
- `crestodian/rescue-pending/*.json`
- `openclaw/rescue-pending/*.json`
- `plugins/phone-control/armed.json`
- `.openclaw-wiki/log.jsonl` de Memory Wiki
- `.openclaw-wiki/state.json` de Memory Wiki
- `.openclaw-wiki/locks/` de Memory Wiki
- `.openclaw-wiki/source-sync.json` de Memory Wiki
- `.openclaw-wiki/import-runs/*.json` de Memory Wiki
- `.openclaw-wiki/cache/agent-digest.json` de Memory Wiki
- `.openclaw-wiki/cache/claims.jsonl` de Memory Wiki
- `.clawhub/lock.json` de ClawHub
- `.clawhub/origin.json` de ClawHub
- `.openclaw-profile-decorated` de decoración de perfiles del navegador
- iniciadores de sesiones respaldados por archivos `SessionManager.open(...)`
- fachadas de listado de transcripciones `SessionManager.listAll(...)` y `TranscriptSessionManager.listAll(...)`
- fachadas de bifurcación de transcripciones `SessionManager.forkFromSession(...)` y
  `TranscriptSessionManager.forkFromSession(...)`
- fachadas de sustitución de sesiones mutables `SessionManager.newSession(...)` y `TranscriptSessionManager.newSession(...)`
- fachadas de sesiones de rama `SessionManager.createBranchedSession(...)` y
  `TranscriptSessionManager.createBranchedSession(...)`

La prohibición debe permitir que las pruebas creen accesorios heredados y que el código de migración
lea/importe/elimine fuentes de archivos heredadas. Los archivos auxiliares SQLite no publicados siguen prohibidos
y no reciben permisos de importación de Doctor.

## Criterios de finalización

- Las escrituras de datos y caché en tiempo de ejecución se dirigen a la base de datos SQLite global o del agente.
- El entorno de ejecución deja de escribir índices de sesión, JSONL de transcripciones, JSON del registro de entornos aislados,
  SQLite auxiliar de tareas o SQLite auxiliar del estado de plugins. Se eliminan los importadores SQLite auxiliares no publicados
  de tareas y estado de plugins.
- La importación de archivos heredados es exclusiva de Doctor.
- La copia de seguridad produce un archivo con instantáneas compactas de SQLite y prueba de integridad.
- Los trabajadores de agentes pueden ejecutarse con almacenamiento en disco, almacenamiento temporal VFS o almacenamiento
  experimental exclusivamente VFS.
- La configuración y los archivos explícitos de credenciales siguen siendo los únicos archivos de control persistentes
  no pertenecientes a la base de datos que se esperan.
- Las comprobaciones del repositorio impiden reintroducir almacenes de archivos heredados en tiempo de ejecución.
