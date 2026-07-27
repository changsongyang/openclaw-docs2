---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 'Distribuye sesiones en máquinas efímeras en la nube: aprovisionamiento, entorno de ejecución de trabajadores, inferencia mediante proxy y transmisión de resultados.'
title: Trabajadores en la nube
x-i18n:
    generated_at: "2026-07-26T05:10:09Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

Los trabajadores en la nube permiten que una sesión ejecute el bucle de su agente en una máquina temporal en la nube mientras todo lo relacionado con la sesión permanece donde siempre ha estado: visible en la barra lateral, transmitiéndose en directo y con la transcripción bajo la propiedad del Gateway. El Gateway arrienda una máquina, instala en ella una copia fijada de OpenClaw, sincroniza el espacio de trabajo de la sesión y entrega el bucle del turno a un proceso `openclaw worker` restringido. Las llamadas al modelo se canalizan de vuelta a través del Gateway, por lo que las credenciales del proveedor nunca salen de la máquina local, y el almacenamiento en caché de prompts sigue funcionando porque el proveedor recibe un flujo continuo.

Cuando finaliza el trabajo (o la máquina deja de funcionar), la máquina se descarta. El estado duradero —la transcripción, los commits del espacio de trabajo y los registros de ubicación— reside en el Gateway.

<Note>
Los trabajadores en la nube son opcionales e invisibles hasta que se configura un perfil. Las instalaciones sin configurar no muestran nuevos RPC, opciones de configuración ni elementos de interfaz.
</Note>

## Qué se ejecuta en cada lugar

| Aspecto                                                 | Ubicación                                                                         |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Bucle del agente + herramientas (`exec`, `read`, `write`, `edit`, …) | Máquina del trabajador en la nube                                                                 |
| Inferencia del modelo y credenciales del proveedor                | Gateway (canalizadas mediante la referencia `{provider, model}`)                               |
| Transcripción (duradera, almacén de sesiones)                     | Gateway                                                                          |
| Transmisión en directo a la barra lateral                         | Distribución del Gateway, alimentada por el flujo de eventos reproducible del trabajador                      |
| Historial de Git del espacio de trabajo                                   | Creado en la máquina sin credenciales; el Gateway adopta los commits y se encarga del push/PR |

La máquina no necesita puertos entrantes salvo `sshd`: el Gateway establece una conexión saliente mediante SSH fijado y un túnel inverso transporta de vuelta el WebSocket del trabajador. El proveedor Crabbox incluido fuerza la ruta SSH pública y desactiva la inscripción administrada en Tailscale. El acceso saliente a Internet depende de la política del proveedor; el perfil predeterminado de AWS puede acceder a Internet a menos que se restrinjan su red o su grupo de seguridad.

## Requisitos

- Un Plugin de proveedor de trabajadores. El Plugin `crabbox` incluido controla la CLI de [Crabbox](https://github.com/openclaw/crabbox), que intermedia los arrendamientos entre backends en la nube (AWS, Hetzner y otros). El binario `crabbox` debe estar en `PATH` (o se debe establecer `settings.binary`) y las credenciales del proveedor ya deben estar configuradas. La admisión de AWS requiere Crabbox 0.38.1 o posterior.
- Para los trabajadores de Crabbox en AWS, el valor efectivo de `aws.instanceProfile` debe estar vacío. El proveedor comprueba `crabbox config show --json` antes de la asignación y, después, exige que `crabbox inspect --json` informe de `providerMetadata.instanceProfileAttached: false` desde `DescribeInstances` de EC2. Los arrendamientos que tengan un rol de instancia o carezcan de metadatos autoritativos se detienen y rechazan.
- Node.js en la máquina arrendada. Las imágenes básicas de nube normalmente no lo incluyen; instálelo mediante el comando `setup` del perfil.
- Una sesión con un worktree administrado y propiedad de la sesión (créelo con `worktree: true`). El envío traslada el contenido de ese worktree; los directorios normales se sincronizan como un espejo del manifiesto.

## Configuración

Añada un perfil bajo `cloudWorkers.profiles` en `openclaw.json`:

```json
{
  "cloudWorkers": {
    "profiles": {
      "aws": {
        "provider": "crabbox",
        "install": "bundle",
        "settings": {
          "provider": "aws",
          "class": "standard",
          "ttl": "8h",
          "idleTimeout": "45m",
          "setup": "test -x /usr/bin/node || (curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash - && sudo apt-get install -y nodejs)"
        }
      }
    }
  }
}
```

Campos del perfil:

| Clave        | Significado                                                                                                                                                                                                                                        |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | Id. del proveedor de trabajadores registrado por un Plugin (`crabbox` para el Plugin incluido).                                                                                                                                                                  |
| `install`  | `bundle` (valor predeterminado) distribuye la compilación del Gateway en ejecución; `npm` instala la versión publicada exacta del Gateway con integridad fijada. `npm` requiere que el Gateway se ejecute desde una versión empaquetada.                                                      |
| `settings` | JSON propiedad del proveedor. Para crabbox: `provider` (backend), `class` (clase de máquina), `ttl`, `idleTimeout` (duraciones de Go), y opcionalmente `setup` y la ruta absoluta `binary`. OpenClaw fuerza el SSH público y desactiva Tailscale administrado para estos arrendamientos. |
| `lifetime` | Política almacenada opcional (`idleTimeoutMinutes`, `maxLifetimeMinutes`).                                                                                                                                                                           |

### El comando de preparación

`settings.setup` se ejecuta en la máquina arrendada después de que SSH esté listo y antes de instalar OpenClaw. Se ejecuta en **cada** intento de aprovisionamiento (incluidas las repeticiones tras un envío interrumpido), por lo que debe ser idempotente: proteja las instalaciones con una comprobación `command -v`/`test -x`, como en el ejemplo. Si la preparación falla, el proveedor detiene el arrendamiento y el envío se cierra de forma segura; no queda ninguna máquina parcialmente configurada en ejecución.

### Canales de instalación

- **`bundle`** empaqueta el `dist` del Gateway en ejecución, un `package.json` depurado y todos los paquetes del espacio de trabajo a los que hace referencia la compilación, todo ello protegido por un hash de contenido. La máquina verifica el paquete intacto con ese hash y, después, instala las dependencias npm de producción (con los scripts desactivados). Así se ejecuta una compilación de desarrollo en un trabajador.
- **`npm`** demuestra que la versión existe en el registro público, fija su integridad SHA-512 e instala `openclaw@<version>` de modo que coincida exactamente con el Gateway.

## Envío de una sesión

En la interfaz de control, abra **Nueva sesión**, elija un agente cuyo entorno de ejecución configurado sea OpenClaw, seleccione un destino **Nube · perfil** configurado en el menú **Ubicación** e inicie la tarea. La selección de la nube activa automáticamente el worktree administrado requerido; el Gateway crea la sesión, completa el envío y solo entonces envía el primer turno. La insignia del servidor en la barra lateral de la sesión muestra el estado duradero de la ubicación. Los destinos en la nube no se ofrecen para catálogos de sesiones de CLI externas.

El flujo RPC equivalente es:

Cree una sesión con un worktree administrado y, después, envíela (el RPC requiere `operator.admin` y solo existe cuando hay perfiles configurados):

Los trabajadores en la nube ejecutan el entorno de ejecución del agente de OpenClaw. Elija un `openai/*` u otro modelo que se resuelva en ese entorno de ejecución; las sesiones configuradas para un entorno de ejecución de CLI externo, como `claude-cli`, no se pueden enviar.

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch` cierra la admisión de turnos locales, espera a que finalice el trabajo activo, aprovisiona el arrendamiento, ejecuta la preparación, inicializa OpenClaw, sincroniza el espacio de trabajo y retorna cuando la ubicación alcanza la propiedad del trabajador `active`. Reserve varios minutos para el primer envío; los arrendamientos y las instalaciones se almacenan en caché cuando el proveedor lo admite. Después, interactúe con la sesión de la forma habitual: los turnos se enrutan automáticamente al trabajador.

Los turnos completados por el trabajador vuelven a conciliar los archivos aptos y con tamaño limitado del espacio de trabajo en el worktree administrado de la sesión antes de liberar la reclamación del turno. El evento terminal del trabajador crea una barrera duradera de resultado pendiente antes de confirmarse. A continuación, el Gateway prepara el resultado completo de la nube como una referencia de Git bajo `refs/openclaw/worker-results/` antes de aplicarlo, de modo que la versión en la nube siga siendo recuperable aunque el Gateway se detenga durante la aplicación. Los resultados del espacio de trabajo utilizan la semántica de archivos de Git: se conservan los archivos normales, los bits de ejecución, los enlaces simbólicos, las adiciones, los cambios y las eliminaciones, pero no los directorios vacíos ni otros modos de directorio. Los cambios de archivos resultantes permanecen en el worktree administrado para su revisión y commit habituales.

La aplicación utiliza el manifiesto del momento del envío como base de fusión. Se aplican los cambios realizados solo en la nube, se mantienen los cambios realizados solo localmente y, para las rutas modificadas en ambos lados, se aplica una política de fusión a tres bandas que conserva la versión local. Un turno con conflictos aun así finaliza: la transcripción informa del resumen limitado de rutas y de la referencia del resultado preparado, la ubicación expone el mismo conflicto en la interfaz de control y los cambios de la nube sin conflictos permanecen aplicados. El aviso incluye `git show <ref>:<path>` para inspeccionar un archivo presente en la nube y un comando `git checkout <ref> -- <path>` con pathspec literal de nivel superior para obtenerlo desde cualquier directorio del espacio de trabajo. Ejecute los comandos en Bash o zsh (Git Bash en Windows). Si la inspección indica que la ruta no existe, el resultado de la nube la eliminó; compruébelo y elimine manualmente la ruta local conservada. Si checkout informa de una obstrucción de archivo o directorio, mueva o elimine la ruta local que bloquea la operación y vuelva a intentarlo. Si la propia referencia preparada ya no existe, considere obsoleto el aviso y no modifique la ruta local. Las referencias preparadas con conflictos siguen disponibles después de liberar la barrera normal del turno; un resultado limpio posterior elimina el aviso y retira la referencia anterior, mientras que la eliminación explícita de la barrera constituye el límite final de limpieza.

Mientras un resultado con barrera sigue conciliándose, un turno nuevo espera hasta 15 segundos a que se libere la reclamación anterior. Si sigue ocupada, el turno falla con un mensaje procesable que indica que «el resultado del espacio de trabajo del turno anterior en la nube aún se está conciliando» y puede volver a intentarse poco después. Al reiniciarse, la recuperación detecta los resultados pendientes y preparados antes de limpiar las reclamaciones obsoletas, completa o vuelve a intentar su aplicación local y solo reclama los entornos inactivos después de preservar el resultado. El diario de reversión limitado de SQLite permite recuperar una aplicación interrumpida en el sistema de archivos sin repetir las mutaciones ya aceptadas.

Cuando el trabajo haya finalizado y no haya ningún turno en ejecución, abra el menú de la sesión y elija **Detener trabajador en la nube…**. El Gateway realiza una última conciliación del espacio de trabajo antes de destruir el entorno. Una ubicación que ya esté en `draining` o `reconciling` está terminando su desmantelamiento; espere a que su insignia cambie a `reclaimed` antes de eliminar la sesión.

En el caso de un trabajador adjunto averiado o fuera de control, un operador puede llamar a `environments.destroy` con `{ "force": true }` como último recurso. El desmantelamiento forzado marca de forma duradera la ubicación como fallida y abandona cualquier resultado remoto sin conciliar antes de destruir el entorno.

El RPC administrativo equivalente es:

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

La asignación avanza mediante una máquina de estados duradera (`local → requested → provisioning → syncing → starting → active`), por lo que, si el Gateway se reinicia durante el despacho, se realiza la reconciliación en lugar de dejar máquinas filtradas. Si falla un turno del modelo, la asignación activa permanece disponible para volver a intentarlo. En caso de conflictos de rutas del espacio de trabajo, se conserva la versión local, se aplica el resto del resultado de la nube y se mantiene la referencia de la nube preparada para su inspección; otros fallos de reconciliación o del ciclo de vida conservan su barrera de recuperación duradera y la parte final del diagnóstico hasta que la recuperación pueda volver a intentarlo de forma segura o recuperar el entorno.

## Modelo de seguridad

- **Entrada cerrada para los workers.** Los workers se comunican mediante un protocolo dedicado en el socket tunelizado con una lista cerrada de métodos permitidos; un worker no puede invocar RPC del operador.
- **Autoridad de herramientas propiedad del Gateway.** Antes de cada turno, el Gateway aplica las políticas actuales de perfil, proveedor, agente, grupo, remitente, entorno aislado, delegación, herencia y límite de ejecución al catálogo fijo de herramientas de programación del worker. El sobre de inicio solo contiene ese subconjunto final de vocabulario cerrado. Los turnos programados con límites explícitos reutilizan su contexto de grupo propietario de confianza sin enviar esa identidad a la máquina ni volver a aplicar una nueva superposición del remitente. Las herramientas que no pertenecen al catálogo del worker siguen sin estar disponibles; un resultado vacío se ejecuta sin herramientas.
- **Credenciales acuñadas y almacenadas como hash.** Cada despacho acuña una credencial de worker; el Gateway solo almacena su hash. La rotación de credenciales y el cercado por época del propietario garantizan como máximo un propietario activo por sesión: un worker obsoleto que se reconecta queda cercado, nunca se fusiona.
- **Fijación de la clave del host.** El proveedor debe proporcionar la clave de host SSH de la máquina durante el aprovisionamiento; el arranque se conecta mediante fijación estricta y falla de forma cerrada si no está disponible.
- **Sin credenciales permanentes del modelo, la plataforma de desarrollo ni la nube en la máquina.** La autenticación del modelo permanece en el Gateway (la inferencia se transmite mediante la referencia `{provider, model}`), los commits de Git del espacio de trabajo se crean sin credenciales de la plataforma de desarrollo y los metadatos del arrendamiento de AWS de Crabbox se comprueban de forma autoritativa para detectar un rol de instancia antes de la configuración. Los comandos de configuración también deben carecer de credenciales.
- **Tráfico saliente controlado por el proveedor.** El túnel inverso elimina cualquier necesidad de OpenClaw de acceder directamente al modelo, pero OpenClaw no modifica los firewalls del proveedor. Restrinja el tráfico saliente en el proveedor del worker cuando la tarea lo requiera.
- **Transcripciones duraderas y procesadas exactamente una vez.** El worker confirma lotes de transcripciones mediante un protocolo de comparación e intercambio con respecto a la hoja de la sesión; una base obsoleta detiene la ejecución ante el fallo en lugar de duplicar o reorganizar la salida de pago.

## Solución de problemas

- **`sessions.dispatch` es un método desconocido** — no hay `cloudWorkers.profiles` configurados o el invocador carece de `operator.admin`.
- **"Los turnos de workers en la nube requieren el entorno de ejecución de OpenClaw"** — elija un modelo cuyo entorno de ejecución configurado sea OpenClaw. Los entornos de ejecución de CLI externos, como `claude-cli`, no admiten la inferencia del worker.
- **"El arranque del worker requiere Node.js en el host arrendado"** — añada una instalación de Node a `settings.setup` (consulte la información anterior).
- **Falla la acreditación del rol de instancia de AWS** — borre `aws.instanceProfile` (y `CRABBOX_AWS_INSTANCE_PROFILE`, si está establecido). Instale Crabbox 0.38.1 o una versión posterior; los binarios anteriores no exponen el contrato autoritativo `providerMetadata.instanceProfileAttached` necesario para la admisión en AWS.
- **El despacho falla con un error del proveedor** — el registro de asignación y `environments.list` conservan el último error, incluida la parte final de stderr de la configuración o el arranque. Las máquinas se destruyen cuando se produce un fallo, por lo que esa parte final constituye la principal fuente de análisis forense.
- **Tiempo de espera agotado en el cliente durante el despacho** — `openclaw gateway call` tiene un tiempo de espera predeterminado de 10s; proporcione un valor amplio en `--timeout` (el despacho sigue ejecutándose en el servidor en cualquier caso, y un nuevo intento durante el aprovisionamiento se rechaza con `session cannot dispatch from placement provisioning`).
- **Worker recuperado después de actualizar desde una beta de 2026.7.2** — esas versiones beta utilizaban el contrato anterior de inicio de workers. Al reiniciarse, OpenClaw destruye un worker inactivo incompatible, conserva la sesión y el espacio de trabajo, marca la asignación como recuperada y aprovisiona un worker actual en el siguiente despacho o turno. Un worker de una versión beta interrumpido mientras aún se estaba iniciando se marca como fallido después de la limpieza; vuelva a intentar el despacho para aprovisionarlo con el contrato actual.
- **Aviso de conflicto en el espacio de trabajo de la nube** — el turno se completó y conservó la versión local de cada ruta indicada. Use los comandos de referencia preparada del aviso para inspeccionar o adoptar la versión de la nube; no es necesario volver a intentarlo para los cambios sin conflictos, que ya se han aplicado.
- **“El resultado del espacio de trabajo del turno anterior en la nube aún se está reconciliando”** — el Gateway esperó brevemente la barrera duradera del resultado anterior y no pudo adquirir la reclamación de la sesión. Espere a que termine la reconciliación y vuelva a intentar el turno; reiniciar el Gateway es seguro porque la recuperación conserva los resultados preparados antes de recuperar un worker inactivo.
- **Mantenimiento de arrendamientos** — `crabbox list --provider <backend>` muestra los arrendamientos activos; `crabbox stop --provider <backend> --id <lease>` libera uno manualmente. Los arrendamientos inactivos caducan según el valor `idleTimeout` del perfil.

## Contenido relacionado

- [Entorno aislado](/es/gateway/sandboxing) — reducción del radio de impacto de la ejecución local de herramientas
- [CLI de sesiones](/es/cli/sessions) — inspección de las sesiones almacenadas
- [Referencia de configuración](/es/gateway/configuration-reference)
