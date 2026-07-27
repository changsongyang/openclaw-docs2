---
read_when:
    - Está implementando o revisando una fase del rediseño de la incorporación.
summary: Plan de implementación para el rediseño de la incorporación de custodios (documento vivo)
title: Rediseño de la incorporación
x-i18n:
    generated_at: "2026-07-26T05:31:10Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: f892991583d0b77a670e9bf7aa5a0c74af3b3eac9e7b0448706486254eb7e2a0
    source_path: start/onboarding-redesign.md
    workflow: 16
---

# Plan de implementación del rediseño de la incorporación

> **Documento vivo.** Esta página realiza un seguimiento del rediseño de la incorporación del custodio a
> nivel de implementación y se actualiza a medida que se completa cada fase. Cuando se
> fusiona la última fase, esta página se reescribe como guía de incorporación para el usuario y se incorpora a
> la navegación de la documentación. Hasta entonces, no se incluye intencionadamente en `docs.json`.

## Objetivo principal

Un usuario sin conocimientos técnicos escribe `openclaw onboard` (o abre la aplicación) y recibe la bienvenida
de una única presencia conversacional: OpenClaw, el custodio del sistema («custodio» es
solo el nombre interno; el usuario siempre ve «OpenClaw»), que encuentra su IA,
lo configura todo con valores predeterminados anunciados en lugar de preguntas, hace eclosionar a su
agente como un momento visible de creación de identidad y permanece accesible para siempre como
encargado del sistema. Magia de forma predeterminada, un único límite de consentimiento y ningún callejón sin salida.

Principios de diseño (decididos, no volver a debatirlos a la ligera):

- **Los valores predeterminados anunciados con opción de deshacerlos fácilmente** sustituyen las preguntas que bloquean. El único
  requisito estricto es que la inferencia funcione; todo lo demás es una propuesta.
- **La pregunta cero es el límite de consentimiento**: «Acceso completo» (recomendado) significa
  que la detección es silenciosa y automática; «Preguntar primero» supedita toda detección —el análisis de la IA,
  de las aplicaciones y de las fuentes de memoria por igual— a un único
  sí explícito, con una ruta totalmente manual que nunca realiza análisis.
- **La conversación como interfaz de usuario con inteligencia progresiva**: la superficie del custodio
  existe antes de que funcione ninguna IA (diálogo con guion), pasa a estar respaldada por el modelo en el
  momento en que se verifica una ruta y lo anuncia de forma visible. Nunca simula inteligencia:
  la entrada de texto libre antes de que se verifique una ruta recibe un amable «deja que primero
  ponga en marcha mi cerebro».
- **La eclosión es una ceremonia**: el mismo hilo, cambio de avatar; el agente se asigna un nombre
  y elige su propio rostro. El custodio enseña la jerarquía una sola vez: «pregúntame
  por el sistema o simplemente pregunta a tu agente; él me transmitirá la consulta».
- **La confianza se clasifica por niveles según la fuente**: las entradas del catálogo oficial pueden estar preseleccionadas;
  las skills de terceros de ClawHub nunca se preseleccionan, independientemente de la
  clasificación del modelo, y sus etiquetas indican que instalan el código del editor.
- **Las instalaciones configuradas son sagradas**: volver a ejecutar la incorporación es una pasada de
  verificación. Nunca vuelve a aplicar la configuración ni reinicia el servicio Gateway.
- **El terminal es la alternativa, no una pregunta**: se prefiere el panel del navegador
  cuando un Gateway está accesible; nunca se pregunta «¿terminal o navegador?».
- **Los modelos débiles reciben una superficie reducida** (`localModelLean` automático), explicada con
  palabras sencillas, nunca en términos de herramientas, modo de código ni ventanas de contexto.

## Flujo publicado actualmente (después de las fases 1-3)

`openclaw onboard` en una instalación nueva de macOS, ruta ideal: cuatro pulsaciones de Intro en total:

1. Nota de seguridad → una pulsación de Intro para confirmarla (persistente; no vuelve a preguntarse).
2. **Pregunta cero**: «¿Cómo debo configurar todo?» — Acceso completo (recomendado)
   o Preguntar primero. Se conserva como `wizard.accessMode`; las nuevas ejecuciones usan de forma predeterminada la
   opción guardada. La opción protegida + «configurar manualmente» lleva al selector de proveedores sin
   realizar ningún análisis y también omite el análisis de las fuentes de memoria.
3. **Teatro de detección**: detecta las CLI de programación, las claves de entorno y los entornos de ejecución locales;
   hace un comentario cuando encuentra agentes de programación; prueba en vivo los candidatos por orden y
   recopila discretamente los errores en una sola línea de resumen (los detalles están en «Ver otras
   opciones»). La primera ruta que funciona se anuncia como valor predeterminado, con una
   ruta de una sola tecla al selector completo; explorar y omitir conserva la
   ruta funcional.
4. Oferta de importación de memoria (Claude Code / Codex / Hermes), omitida cuando se
   rechazó la detección.
5. Solo en instalaciones nuevas: el plan de configuración estándar se aplica automáticamente
   (espacio de trabajo, servicio Gateway, sesiones: el mismo plan que ejecuta el «sí»
   conversacional). Las instalaciones configuradas muestran «ya está configurado» y nunca modifican el
   servicio.
6. **Recomendaciones de aplicaciones**: aplicaciones instaladas que el modelo verificado hace coincidir
   con los catálogos oficiales + ClawHub; los plugins de canales oficiales aparecen
   premarcados, mientras que las skills de terceros requieren aceptación y muestran una etiqueta de advertencia. Se puede omitir;
   interruptor de desactivación `wizard.appRecommendations`.
7. **Eclosión**: cuando un Gateway está accesible, la transferencia al navegador se abre (GUI) o
   muestra (sin interfaz gráfica/SSH) la URL del panel y espera a que la interfaz de usuario de Control
   se conecte: «Panel conectado; se continuará en el navegador». De lo contrario, o
   con `--tui`, se abre la TUI del terminal, inicializada con el mensaje de eclosión de arranque,
   y el agente se presenta.

La incorporación con Gateway remoto conserva su transferencia conversacional heredada
(`handoffMode: "chat"`); la configuración debe aplicarse en el Gateway remoto.

## Fases

| #   | Fase                                                                                                                                                     | Superficie              | Estado                                                                                                                            |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Recomendaciones de plugins de aplicaciones instaladas (análisis, candidatos, comparador de IA, paso del asistente, comando de Node `device.apps`)                                              | CLI clásica + guiada | fusionada ([#109668](https://github.com/openclaw/openclaw/pull/109668))                                                              |
| 2   | Estructura principal del custodio en la CLI (pregunta cero, teatro de detección, aplicación automática + eclosión)                                                                                | CLI guiada           | fusionada ([`a83ed13204f1`](https://github.com/openclaw/openclaw/commit/a83ed13204f118adf1009e5ac88d5afe1905b86c))                   |
| 3   | Transferencia prioritaria al navegador (detección de sesiones de GUI, espera de conexión al panel, TUI como alternativa)                                                                | CLI → web            | fusionada ([#110054](https://github.com/openclaw/openclaw/pull/110054))                                                              |
| 4   | Superficie web del custodio (tarjetas de opciones, campo `question` con tipo en `openclaw.chat`, reflejo de los pasos del asistente, transferencia de la primera ejecución)                                 | Interfaz de usuario de Control           | fusionada ([#110141](https://github.com/openclaw/openclaw/pull/110141), [#110242](https://github.com/openclaw/openclaw/pull/110242)) |
| 5   | Eclosión y arranque (almacén de recomendaciones con semántica de una sola vez, secuencia de nacimiento con asignación automática de nombre, transferencia automática a la eclosión tras una configuración nueva; jerarquía de avatares aplazada) | arranque del agente      | fusionada ([#110173](https://github.com/openclaw/openclaw/pull/110173), [#110331](https://github.com/openclaw/openclaw/pull/110331)) |
| 6   | Presencia del custodio PR1 (entrada fijada en la barra lateral, Preguntar a OpenClaw en Configuración, saludo del encargado con la interfaz normal; los comentarios de eventos y la invocación desde canales quedan para PR2)    | web + canales       | fusionada ([#110269](https://github.com/openclaw/openclaw/pull/110269))                                                              |
| 7   | Resiliencia (custodio accesible con una configuración dañada, recuperación parcial de superficies, corrección automática)                                                                   | Gateway              | seguimiento                                                                                                                         |

## Notas de implementación por fase

### Fase 1 — recomendaciones de aplicaciones (PR #109668)

- Analizador: `src/infra/installed-apps.ts` (enumeración de macOS sin TCC; sigue
  paquetes `.app` enlazados simbólicamente).
- Candidatos: catálogos oficiales + búsqueda en ClawHub, presupuesto total de 20 s, degradación
  sin conexión elegante a candidatos únicamente del catálogo. Las entradas del catálogo son manifiestos de paquetes
  sin un `id` de nivel superior; los candidatos se identifican mediante el id del
  plugin resuelto (probado contra regresiones con los catálogos incluidos reales; la identificación mediante
  `entry.id` llegó a contraer todo el catálogo y descartar todas las recomendaciones
  oficiales).
- Comparador de IA: una finalización en la ruta verificada
  (`src/system-agent/setup-app-recommendations.ts`); sin mapa seleccionado de identificadores de paquetes:
  el modelo rechaza coincidencias de nombres fortuitas. La salida está limitada por el
  propio presupuesto `maxTokens` del modelo resuelto (la capa de transmisión lo aplica cuando no se
  proporciona ningún límite explícito).
- **Protección de la cadena de suministro**: el texto de los listados de ClawHub lo controla el editor y
  llega al prompt del comparador, por lo que un listado puede promocionarse a sí mismo como
  «recomendado». Solo se pueden preseleccionar las entradas del catálogo oficial; las skills de ClawHub
  siempre requieren una marca explícita y se etiquetan como «skill de ClawHub de terceros;
  instala el código de su editor».
- Comando de Node `device.apps` (host de Node TS, paridad del sobre de Android), uso compartido
  desactivado de forma predeterminada; interruptor de desactivación del Gateway `wizard.appRecommendations`.
- La entrega reside en el asistente clásico y en el flujo guiado del custodio
  (`src/wizard/setup.app-recommendations.ts`); la redirección a la parte final del
  arranque queda para la fase 5 (el servicio ya acepta una fuente de inventario
  inyectable). La semántica de una sola vez (ofrecer solo hasta que se acepte, análisis almacenado) también se incorpora
  con el almacén de la fase 5; actualmente, una nueva ejecución vuelve a ofrecerlo.
- También se corrigió: los prompts personalizados `completeSetupInference` ya no heredan el
  límite de salida de 32 tokens de la sonda de verificación (`SETUP_INFERENCE_TEST_MAX_TOKENS`
  solo se aplica a la sonda «responder OK»).

### Fase 2 — estructura principal del custodio en la CLI (PR #109841)

- Reestructuración del flujo en `src/commands/onboard-guided.ts`; la incorporación con Gateway remoto
  conserva su transferencia de chat heredada mediante `handoffMode: "chat"`.
- La pregunta cero conserva `wizard.accessMode` ("full" | "guarded"); las nuevas ejecuciones
  usan de forma predeterminada la opción guardada (aceptar el valor predeterminado nunca puede
  rebajar silenciosamente el modo protegido a completo). El modo protegido + manual usa
  `listManualSetupInferenceOptions` (solo configuración/manifiestos, sin sondeo) y
  omite el análisis de fuentes de memoria.
- Detección: recopilación silenciosa de errores (una sola línea de resumen; detalles en
  «Ver otras opciones»), comentario sobre agentes de programación y valor predeterminado de ruta anunciado. Los
  recuentos de sesiones del comentario se aplazan (por ahora solo son cualitativos) hasta que exista una
  interfaz económica para contar sesiones.
- Instalaciones nuevas: `applySystemAgentSetup` (el «sí» conversacional determinista),
  seguido de la eclosión mediante `launchTuiCli`, inicializada con el mensaje de arranque.
  Instalaciones configuradas (modelo o configuración de Gateway preexistentes; las marcas de tiempo
  del asistente no demuestran nada, pues se comparten con la configuración y el doctor):
  solo verificación, sin aplicación ni reinicio del servicio Gateway. Un fallo de aplicación
  vuelve al chat conversacional.

### Fase 3 — transferencia prioritaria al navegador (PR #110054, fusionada)

- `src/commands/onboard-browser-handoff.ts` se encarga exclusivamente de la detección de sesiones gráficas
  (`SSH_CONNECTION`/`SSH_TTY`; `DISPLAY`/`WAYLAND_DISPLAY` en Linux)
  y de la espera de 60 segundos para la GUI / 300 segundos para SSH. Actualmente, la incorporación guiada
  habilita la transferencia solo en macOS; `--tui` y otras plataformas mantienen la
  salida al terminal. La habilitación en Linux/Windows queda como seguimiento.
- Los enlaces del panel usan los mismos auxiliares `resolveAdvertisedControlUiLinks`,
  `resolveLocalControlUiProbeLinks` y `buildOnboardingControlUiUrl`
  que la finalización clásica. El inicio del navegador usa el auxiliar compartido `openUrl`.
- La comprobación de disponibilidad consulta periódicamente el RPC `system-presence` existente como **cliente de bucle invertido en modo CLI
  que presenta el secreto compartido configurado**: la ruta de confianza que utiliza cada
  comando `openclaw`. Un cliente de la interfaz de control con autenticación compartida sin procesar se rechaza
  con "se requiere la identidad del dispositivo" en gateways SecretRef. La comprobación previa de accesibilidad
  resuelve el mismo destino (y secreto) que el bucle de espera, por lo que la
  puerta y la espera nunca pueden discrepar sobre la autenticación. La transferencia solo se completa
  cuando una fila de presencia `openclaw-control-ui`/`webchat` conectada es nueva
  respecto al estado de referencia anterior al inicio (un panel que ya estuviera abierto no puede
  completarla).
- `gateway.controlUi.enabled: false` interrumpe el proceso antes de mostrar cualquier URL.
- Comprobado de extremo a extremo con un gateway aislado que usa la misma configuración: impresión de la URL → conexión de un
  navegador real → "Panel conectado: continúe en el navegador" → sin
  salida al terminal. Un bloqueo anterior por "discrepancia de token" era un artefacto del
  entorno de pruebas; consulte el manual de pruebas a continuación.

### Fase 4 — superficie web del custodio (fusionada: #110141, #110242)

- Página `/custodian` sobre `openclaw.chat` con el componente de tarjetas de opciones
  (2-4 tarjetas, como máximo una recomendada, siempre se puede omitir); marco de incorporación mediante
  `?onboarding=1`; la finalización de la configuración inicial del modelo transfiere el control a esta página.
- Las preguntas estructuradas son un campo aditivo tipado `question` en
  `SystemAgentChatResult` (texto `reply` por opción; la prosa siempre aparece por separado
  para la aplicación de macOS/TUI). Productores: ambas variantes de bienvenida de la incorporación y
  los pasos de selección/confirmación del asistente alojado con 2-4 opciones cerradas; los asistentes de canales
  reales se muestran como tarjetas. Se eliminó la solución provisional de marcadores de cadena de PR1.
- La propiedad de la sesión se limita a la URL del gateway + cada credencial presentada
  (token, contraseña, token de arranque, token de dispositivo almacenado, persistente durante
  interrupciones transitorias del saludo); los turnos fallidos del usuario nunca pueden reproducirse; la entrada
  confidencial se envía literalmente y se oculta en la transcripción.

### Fase 5 — salida y arranque (fusionada: #110173, #110331)

- El custodio crea un agente sin nombre (llamada a herramienta); el arranque del agente comienza
  asignándose un nombre. PR1 incluye la ceremonia limitada a tres pasos (nombre → frase del alma
  → pregunta sobre Skills) y aplaza la secuencia de avatar autodibujado/generación de imágenes
  (candidatos generados por el modelo → marcas predefinidas → conservar el logotipo) para un seguimiento. Mismo
  hilo, cambio de avatar; la marca de la garra queda reservada para el custodio. La
  identidad acordada se conserva dos veces: en `IDENTITY.md`/`SOUL.md` (lo que lee el agente)
  y mediante `openclaw agents set-identity` (lo que muestran los canales y la interfaz
  de usuario).
- Las recomendaciones (servicio de la fase 1, análisis almacenado con semántica de una sola ejecución) llegan como
  último paso del arranque antes de eliminar el archivo de arranque: "¿conjunto mínimo
  o máxima comodidad?" El arranque lee la oferta almacenada mediante
  `openclaw onboard recommendations --json` (solo identificadores opacos de instalación) y
  la confirma después de procesar la elección, para no volver a preguntar nunca. Los botones de
  conexión de canales incluyen manuales de configuración específicos de cada canal; el agente recopila
  las credenciales mediante conversación y transmite las escrituras de configuración al custodio
  ("preguntando a OpenClaw…" es la fórmula canónica).
- El autoaprendizaje se pregunta, no se anuncia, y también sirve como consentimiento para el taller de Skills;
  describa la confianza en las versiones de ClawHub, el análisis, la verificación y las comprobaciones de
  integridad, además de la advertencia sobre el código del editor; nunca dé a entender que todas las versiones están firmadas.
- Se publicó la salida automática: la aplicación de la configuración de una instalación nueva anuncia la salida y
  transfiere el control (TUI del terminal / `open-agent` para clientes del gateway); la página web
  llega al chat del agente con el borrador "¡Despierta, amigo mío!" rellenado previamente. La
  transferencia solo se activa tras una verificación correcta posterior a la escritura. Ofrecer la opción cuando haya
  cero agentes después de una eliminación (en lugar de hacerlo automáticamente) queda como mejora de seguimiento.

### Fase 6 — presencia del custodio (PR1 fusionada: #110269; los comentarios/la invocación corresponden a PR2)

- Publicado en PR1: entrada "OpenClaw" fijada de forma predeterminada en la barra lateral (perfiles nuevos;
  los usuarios existentes conservan los elementos fijados y pueden acceder a ella mediante personalización/Más), "Preguntar a
  OpenClaw" como primera entrada de Ajustes y visitas `/custodian` con el marco normal
  que solicitan el saludo del cuidador (sin variante de bienvenida de la incorporación), con
  Salir de la configuración visible solo en el modo de incorporación. Un panel de Ajustes insertado y acoplado
  necesita extraer la vista de conversación compartida (seguimiento).
- Comentarios reactivos a eventos con mecanismos de protección contra comportamientos intrusivos: solo cambios importantes o
  fallidos, como máximo una vez por visita a los ajustes, salvo que se soliciten. El mismo
  punto de integración de eventos permite que el custodio sea posteriormente la voz para una autenticación degradada o canales
  averiados.
- Canales: invisibles en el uso diario (el agente actúa como intermediario); accesibles mediante una
  invocación explícita y ante eventos de agente inactivo en el mismo hilo, con su propio nombre y
  avatar de garra cuando la plataforma lo permita.
- Modelo débil detectado durante la configuración: se establece automáticamente `localModelLean` y el custodio
  lo comunica con palabras claras y ofrece una actualización.
- El custodio conoce su apodo interno ("algunos me llaman el
  custodio; OpenClaw está bien") y siempre se refiere al agente por su nombre.

### Fase 7 — resiliencia (se necesita una decisión del responsable antes de desarrollarla)

El planteamiento original —"el custodio debe ser accesible por muy dañada que esté
la configuración"— entra en conflicto con la política de seguridad del repositorio: la guía raíz
indica que el Gateway **rechaza iniciarse** cuando la configuración no es válida estructuralmente,
y solo los fallos del responsable de SecretRef se degradan a capacidades configuradas como no disponibles.
Proporcionar cualquier superficie desde una configuración no válida supone un cambio de política,
no un detalle de implementación. Hay dos alcances; elija uno:

- **Opción A (recomendada, conforme con la política): diagnóstico y reparación automáticos desde la CLI.** Cuando el
  inicio de un gateway o de la CLI falla debido a una configuración no válida de formato conocido, la CLI ofrece
  ejecutar `openclaw doctor --fix` (o lo ejecuta con consentimiento), vuelve a intentarlo una vez e
  informa con claridad. El comportamiento del gateway no cambia; el custodio permanece accesible
  mediante la ruta de SecretRef degradada existente y el terminal.
- **Opción B (requiere la aprobación explícita del responsable y una revisión de seguridad): modo de
  superficie mínima del gateway.** Ante una configuración no válida estructuralmente, se inicia una
  superficie restringida que solo proporciona la conversación con el custodio y las acciones de diagnóstico. Esto
  modifica el contrato de inicio con cierre seguro y debe definir su propia estrategia de
  protección de entrada antes de escribir código.

Seguimientos pendientes de las fases 4-6 (registrados, sin programar): secuencia de avatar/generación de imágenes
para la salida; representación en la aplicación de macOS del campo tipado `question`; un
panel de Ajustes insertado y acoplado para el custodio (necesita extraer la vista de conversación
compartida); comentarios reactivos a eventos e invocación desde canales/recuperación ante agentes inactivos
(PR2 de la fase 6); `localModelLean` automático para modelos débiles; decidir si los elementos
fijados en la barra lateral de los usuarios existentes deben adoptar la entrada OpenClaw.

## Manual de pruebas y publicación (aprendido con esfuerzo; léase antes de las fases 4-6)

- **`OPENCLAW_STATE_DIR` no aísla el servicio del Gateway.** La
  etiqueta de LaunchAgent (`ai.openclaw.gateway`) es global para la máquina: una prueba de incorporación
  de instalación nueva con un directorio de estado aislado REESCRIBIRÁ y REINICIARÁ el servicio real
  de la máquina (los scripts envoltorio se guardan dentro del directorio aislado; el siguiente inicio
  del servicio falla cuando se limpia ese directorio). Después de cualquier prueba de instalación nueva,
  restaure con `openclaw gateway install --force && openclaw gateway
restart` desde el entorno real y verifique el plist. Seguimiento del producto:
  etiquetas de servicio limitadas al directorio de estado o detección de un servicio ajeno durante la incorporación.
- **Entorno seguro de extremo a extremo**: rellene previamente la configuración aislada con una sección `gateway`
  (para que la incorporación siga la ruta de instalación configurada y nunca toque
  el servicio) y ejecute `openclaw gateway run` como un proceso normal en primer plano en
  un puerto libre con un token simple. Este entorno demostró el bucle de la fase 3,
  incluida la conexión de un navegador real.
- **Las rutas de autenticación difieren según la identidad del cliente, no solo según las credenciales.** Las consultas de presencia y
  otras lecturas del operador usan un cliente de bucle invertido en modo CLI con las credenciales de la
  misma configuración. Los gateways con autenticación mediante token requieren el secreto compartido; los gateways
  SecretRef/sin autenticación pueden recurrir a la autenticación de bucle invertido de confianza sin token. Un cliente
  de navegador identificado como interfaz de control necesita la identidad del dispositivo o la concesión de bucle invertido
  de contexto seguro. Una sonda que se autentica en un gateway que proporciona una
  configuración DIFERENTE (consulte el problema de LaunchAgent) falla con "discrepancia de token"; ese
  artefacto bloqueó brevemente la fase 3.
- **Sondas de finalización**: `runSetupInferenceTest` limita la sonda de verificación a
  32 tokens de salida; los prompts personalizados eluden el límite y quedan restringidos por
  el propio `maxTokens` del modelo. Los modelos de razonamiento consumen primero ese presupuesto con razonamiento
  oculto; un turno sin texto suele significar que el presupuesto se agotó en esa fase.
- **La publicación del agente necesita CI alojada para el encabezado exacto.** Es posible que el flujo de trabajo pesado `CI`
  no se ponga en cola en los envíos con mucha carga de la organización; la alternativa para el responsable es
  ejecutar una puerta de publicación en la rama del pull request:

  ```bash
  gh workflow run ci.yml --ref <branch> -f target_ref=<head-sha> -f release_gate=true -f pull_request_number=<pr>
  ```

  La ejecución debe realizarse en la
  referencia de la rama para que `head_sha` coincida, y el título pasa a ser
  `CI release gate <sha>`, que `scripts/verify-pr-hosted-gates.mjs`
  acepta. Después, `scripts/pr` prepara/fusiona de la forma habitual.

- **Puertas que la CI aplica además de las pruebas específicas**: mapa de documentación
  (`pnpm docs:map:gen` después de añadir cualquier página de documentación), oxlint (`no-map-spread`,
  `max-lines`: divida los archivos, nunca suprima), `check:test-types`, código muerto de knip
  (exporte solo lo que consuma el código de producción; dirija las pruebas mediante API públicas)
  y el clasificador de particiones de pruebas en vivo
  (`test/scripts/test-live-shard.test.ts` debe enumerar cualquier `*.live.test.ts` nuevo).

## Registro de decisiones

- Escaneo mágico con interruptor de desactivación, no basado primero en el consentimiento (fase 1; la salida persistente
  informa del uso del modelo y de ClawHub antes del escaneo, y la nota de resultados lo reitera).
- Flujo vertical completo, incluido el comando `device.apps` del Node (fase 1).
- Las Skills de terceros de ClawHub nunca están preseleccionadas y se indica que
  instalan el código del editor; las entradas oficiales pueden estar premarcadas
  (fase 1, postura de seguridad implementada).
- Dos tarjetas de acceso, no tres; el consentimiento se presenta al principio de la elección (fase 2).
- Eclosión automática con anuncio, no un botón bloqueante (fases 2/5).
- Primero el navegador: la eclosión en el terminal es la alternativa, nunca una pregunta «¿terminal o
  navegador?» (fase 3).
- El custodio obtiene presencia en el canal (invocación + recuperación), no solo en la web/CLI
  (fase 6).
- La eclosión ocurre en el mismo hilo con un cambio de avatar; al completarse,
  la aplicación pasa a la interfaz habitual (fase 5).
- La superficie de configuración conserva el nombre «Configuración»; el custodio está allí
  (y en la barra lateral) en lugar de sustituirla (fase 6).
- Las tarjetas de opciones están restringidas: de 2 a 4 opciones, exactamente una recomendada y siempre
  se pueden omitir; el mismo componente se utiliza en la incorporación y en la herramienta de preguntas del agente
  (fase 4).
- «Consultando a OpenClaw…» es la fórmula canónica de delegación; las almas pueden añadir matices,
  pero la narración de la herramienta se mantiene sencilla (fase 5).
- El texto dirigido al usuario nunca dice «modo de código», «herramientas» ni «ventana de contexto» al
  explicar el recorte para modelos de baja capacidad (fase 6).

## Limitaciones conocidas y tareas de seguimiento

- La etiqueta de LaunchAgent no está limitada al directorio de estado (problema de pruebas mencionado anteriormente; también es una
  limitación real del producto para varias instancias).
- Semántica de ejecución única de las recomendaciones y el escaneo almacenado (fase 5); las repeticiones
  actualmente vuelven a ofrecerlas.
- La transferencia al navegador solo está disponible en macOS; la compatibilidad con Linux/Windows está pendiente.
- La ocurrencia sobre el número de sesiones es cualitativa; los recuentos necesitan un mecanismo sencillo para contar sesiones.
- La transferencia al navegador llega al panel habitual; el enlace profundo al custodio en modo de incorporación
  llegará con la fase 4.
