---
read_when:
    - Está verificando el cambio al almacenamiento SQLite de la Ruta 3 en un Gateway activo
    - Es necesario distinguir las divergencias esperadas del formato JSONL heredado de los fallos en tiempo de ejecución
    - Está creando o revisando el entorno E2E de SQLite en vivo controlado por agentes
summary: Diseño para la prueba en vivo del Gateway del cambio de sesión/transcripción de SQLite de la Ruta 3
title: Entorno de pruebas E2E de SQLite en vivo de la ruta 3
x-i18n:
    generated_at: "2026-07-26T05:28:48Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2749bf47cb4967bc80a5ed37a12f2a553f3b388ed8cd90cfb3217e1b5e8afae9
    source_path: reference/path3-live-sqlite-e2e-harness.md
    workflow: 16
---

El arnés E2E de SQLite en vivo de la Ruta 3 demuestra que el Gateway utiliza SQLite como
almacén canónico de sesiones y transcripciones, mientras que los archivos JSONL heredados siguen siendo
datos de entrada para la migración o material de archivo. Es un arnés de prueba para mantenedores, no una
herramienta de diagnóstico normal para usuarios.

Después de que un Gateway haya procesado tráfico posterior a la migración, la paridad con los archivos JSONL heredados deja de
ser una señal válida del estado del entorno de ejecución. Un Gateway migrado y en buen estado puede tener
filas de transcripción de SQLite cuyos recuentos difieran de los de los archivos JSONL heredados, porque los nuevos turnos
solo deben actualizar SQLite. Por tanto, el arnés en vivo debe medir el
comportamiento del Gateway, los cambios en las filas de SQLite, la inactividad de los archivos heredados y el estado de los registros en cada
paso.

## Forma del comando

El comando en vivo previsto es:

```bash
node scripts/path3-live-sqlite-e2e.mjs \
  --url http://127.0.0.1:18789 \
  --agent main \
  --session-key agent:main:path3-live-e2e:<timestamp> \
  --json
```

El comando se conecta a un Gateway que ya está en ejecución. No inicia, detiene,
importa ni vuelve a ejecutar la migración, salvo que posteriormente se añada un modo de migración
explícito. Una variante para CI o un entorno local aislado puede utilizar
`test/helpers/openclaw-test-instance.ts`, pero la ruta de prueba en vivo debe inspeccionar
el Gateway real del operador y su base de datos SQLite real por agente.

## Prueba aislada de la CLI compilada

El ejecutor de pruebas de la CLI compilada prepara un almacén de sesiones heredado aislado, inicia el
Gateway recompilado y demuestra que, al arrancar, importa las sesiones heredadas activas en
SQLite antes de que comiencen las lecturas del entorno de ejecución. No debe ejecutar `openclaw doctor --fix`
antes del primer inicio del Gateway, ya que eso demostraría la ruta de migración manual
en lugar de la ruta de actualización que reciben los usuarios en el primer arranque tras el cambio.

Después de la importación durante el arranque, la prueba aislada puede ejecutar
`openclaw doctor --session-sqlite inspect` y
`openclaw doctor --session-sqlite validate` como evidencia de diagnóstico. Esos
comandos de doctor no controlan la migración en la prueba de actualización durante el arranque.
Los escenarios independientes de importación mediante doctor deben preparar archivos de transcripción heredados junto con
archivos auxiliares de trayectoria y verificar que doctor archive esos artefactos mientras SQLite
sigue siendo canónico.

## Comprobaciones previas

Las comprobaciones previas recopilan una referencia inicial y fallan antes de enviar un turno de prueba si el
Gateway no se puede utilizar:

- `GET /health` y el estado detallado del Gateway deben indicar que el
  Gateway está en ejecución y es accesible.
- Las versiones de la CLI y el Gateway deben coincidir con la rama sometida a prueba.
- El arnés registra un cursor de registro para el archivo de registro activo del Gateway.
- El arnés registra los recuentos de las tablas SQLite por agente para `sessions`,
  `session_entries`, `transcript_events`, `transcript_event_identities` y
  `session_routes`.
- El arnés registra `mtime`, `size` y la existencia de los elementos heredados
  `sessions.json`, los archivos JSONL referenciados y las posibles rutas JSONL
  de la sesión de prueba.
- `lsof -p <gateway-pid>` debe mostrar identificadores abiertos de DB/WAL/SHM de SQLite y ningún identificador abierto activo de
  `.jsonl` ni `sessions.json`.

`openclaw doctor --session-sqlite validate` tiene carácter meramente informativo en el modo en vivo.
Después del tráfico posterior al cambio, puede informar de la divergencia esperada respecto a los archivos heredados. El
arnés debe utilizar la salida de doctor para la clasificación y el inventario de migración,
no como criterio de aprobación o fallo del entorno de ejecución.

## Escenario controlado por el agente

El escenario en vivo utiliza una clave de sesión dedicada a la prueba y controla el Gateway
mediante rutas RPC públicas siempre que sea posible. Un turno del agente debería bastar para
ejercitar la persistencia habitual, pero la prueba completa debe cubrir los puntos de integración
de 3.1b que anteriormente requerían comprobaciones en vivo individuales:

- Turno de chat habitual: crear o reutilizar la sesión de prueba, enviar una instrucción real al
  agente, esperar el resultado final del asistente y verificar `chat.history` o una
  proyección equivalente del Gateway.
- Identidad de la transcripción: verificar que el mismo marcador aparezca en el historial del Gateway y en
  las filas de transcripción de SQLite, incluidas las filas con identidades estables de eventos cuando existan.
- Métodos de acceso a los metadatos de sesión: leer la sesión de prueba y determinadas sesiones activas
  existentes mediante los métodos de acceso del Gateway o de sesión y compararlas con las filas de SQLite.
- Proyección de la modificación de sesión: aplicar un cambio reversible en los metadatos del modelo o de la sesión
  de prueba y verificar después que la fila proyectada y la respuesta del Gateway coincidan.
- Ciclo de vida del punto de control de Compaction: enumerar, ramificar y restaurar un punto de control únicamente
  en la sesión de prueba o en una sesión de datos de prueba sintética creada por el arnés.
- Recuperación tras reinicio: ejecutar la ruta segura del marcador de recuperación en una sesión de prueba
  controlada o una instancia de prueba aislada; el modo en vivo solo puede ejecutar este paso cuando
  el conjunto de sesiones de destino sea explícito y reversible.
- Ciclo de vida de la limpieza: eliminar o restablecer la sesión de prueba y verificar después las
  filas del ciclo de vida en SQLite y el estado archivado de la transcripción.

Los puntos de integración específicos del transporte que no puedan ejercitarse con seguridad en el Gateway
en vivo del operador, como la entrada mediante WhatsApp o llamadas de voz, deben utilizar sondeos del entorno de ejecución
a nivel del propietario con el mismo contrato de SQLite, en lugar de simular un transporte externo.

## Aserciones por paso

Cada paso captura el estado anterior y posterior y escribe un registro estructurado de aserciones:

- Los recuentos de filas de SQLite solo avanzan donde se espera.
- Las filas de trayectoria del entorno de ejecución avanzan para las sesiones de prueba respaldadas por marcadores que registran
  eventos del entorno de ejecución.
- La fila de la sesión de prueba tiene los valores esperados de `session_id`, estado, marcas de tiempo,
  metadatos y filas de rutas.
- La proyección del historial o de la sesión del Gateway coincide con la parte final de la transcripción de SQLite.
- No se crea ni modifica ningún archivo JSONL de la sesión de prueba.
- No se crea ningún archivo auxiliar `.trajectory.jsonl`, `.trajectory-path.json` ni
  `trajectory/<session>.jsonl` derivado del marcador para la sesión de prueba.
- Los archivos JSONL heredados existentes y `sessions.json` permanecen sin cambios, salvo que el
  paso sea explícitamente una operación de migración sin conexión o de archivado.
- El proceso del Gateway no abre identificadores de `.jsonl` ni `sessions.json`.
- Los registros desde el cursor anterior no contienen `ERROR`, `FATAL`, `SQLITE_`,
  `no such column`, indisponibilidad del almacén de sesiones, fallo de recuperación tras reinicio ni
  advertencia de conciliación de transcripciones, salvo que el escenario lo incluya explícitamente en la lista de permitidos.

El análisis de los registros forma parte del contrato de aprobación o fallo. Un Gateway que responda a las comprobaciones de estado
pero emita errores de esquema de SQLite o fallos reiterados de conciliación de transcripciones
no se considera correcto para la Ruta 3.

## Artefacto de evidencia

El arnés debe escribir la evidencia en `.artifacts/path3-live-e2e/<timestamp>/`
y mantenerla fuera de git:

- `summary.json`: argumentos del comando, versión del Gateway, resultado, aserción fallida y
  rutas de los artefactos.
- `sqlite-before.json` y `sqlite-after.json`: recuentos de filas y filas de prueba
  seleccionadas.
- `legacy-files.json`: existencia de archivos heredados, `mtime`, tamaño y si cada
  archivo ha cambiado.
- `gateway-log-scan.json`: intervalo del cursor, líneas de registro coincidentes y decisiones sobre la lista de
  permitidos.
- `events.jsonl`: observaciones ordenadas por paso, aptas para los comentarios de prueba del PR.

La prueba del PR debe resumir estos artefactos en lugar de pegar transcripciones completas
o contenido de mensajes privados.

## Reglas de seguridad

- El modo en vivo nunca debe volver a importar archivos JSONL heredados mientras el Gateway esté en ejecución.
- El modo en vivo no debe modificar sesiones que no sean de prueba, salvo para sondeos de reparación
  reversibles y seleccionados explícitamente.
- Cualquier paso destructivo o de migración general requiere una copia de seguridad reciente de la
  base de datos SQLite afectada y del directorio de sesiones heredadas.
- Las copias de seguridad deben limitarse al directorio de la base de datos o de sesiones del agente afectado y reutilizarse
  durante una misma ejecución de prueba para evitar un crecimiento ilimitado del uso del disco.
- El paso de limpieza no debe dejar ninguna sesión de prueba, archivo JSONL de prueba ni archivo heredado
  modificado, salvo que el invocador pase `--keep-artifacts`.

## Resultado satisfactorio

Una ejecución en vivo satisfactoria significa que el Gateway aceptó un flujo de sesión real controlado por el agente,
que todo el estado canónico observado estaba en SQLite, que los archivos heredados del entorno de ejecución permanecieron
inactivos y que los registros se mantuvieron sin errores durante el intervalo medido. No significa
que la paridad con los archivos JSONL heredados permanezca intacta después del tráfico en vivo; se espera que haya divergencias
una vez que SQLite sea el almacén canónico.
