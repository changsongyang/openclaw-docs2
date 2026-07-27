---
read_when:
    - Está implementando clawdbot-d63.2 / clawdbot-04b
    - Está modificando la retención, el restablecimiento, la eliminación o el archivado por eliminación de agentes de sesiones de SQLite
    - Es necesario distinguir las familias de artefactos de la era de SQLite de los archivos auxiliares JSONL heredados.
summary: Plan de la ruta 3 para archivar todos los artefactos de transcripción de SQLite que pertenecen a una sesión
title: Familia de artefactos de sesión SQLite de la ruta 3
x-i18n:
    generated_at: "2026-07-26T04:44:41Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# Familia de artefactos de sesión SQLite de la ruta 3

Esta nota delimita `clawdbot-d63.2`, mientras que `clawdbot-d63.1` se ocupa del auxiliar superpuesto de
archivado para restablecimiento/eliminación en `src/config/sessions/session-accessor.sqlite.ts`.
El archivo de implementación tenía cambios sin confirmar durante esta revisión, por lo que este artefacto registra
el contrato exacto y los puntos de modificación sin interferir con el trabajador paralelo.

## Familia autoritativa

Tras la migración a SQLite, las transcripciones de sesiones activas son filas de SQLite. La familia de
archivado de una sesión es:

- Las filas `transcript_events`, `transcript_event_identities` y `sessions`
  del `sessionId` actual de la entrada.
- El mismo conjunto de filas de transcripción de SQLite para cada `sessionId` al que haga referencia
  `entry.compactionCheckpoints[*].preCompaction.sessionId`.
- El mismo conjunto de filas de transcripción de SQLite para cada `sessionId` al que haga referencia
  `entry.compactionCheckpoints[*].postCompaction.sessionId`.
- El mismo conjunto de filas de transcripción de SQLite para cada `sessionId` de
  `entry.usageFamilySessionIds`.

Archivar únicamente las filas a las que ya no haga referencia ninguna fila
`session_entries` restante ni los metadatos de Compaction o de la familia de uso
de ninguna entrada restante. Esto conserva el estado de ramificación/restauración de puntos de control y de agregación de uso hasta
que desaparezca la última referencia activa.

## Artefactos ajenos a la familia tras la migración

Las variantes generadas de archivos de transcripción por tema y los archivos auxiliares de trayectoria no constituyen estado activo
del entorno de ejecución de SQLite. Son artefactos de archivos heredados:

- Las variantes por tema, como `<sessionId>-topic-<thread>.jsonl`, solo existen para el
  formato de transcripción basado en archivos. SQLite utiliza el identificador de sesión canónico junto con
  los metadatos de entrega de `session_routes`/la entrada en lugar de archivos JSONL por tema.
- Los archivos auxiliares de trayectoria, como `.trajectory.jsonl` y `.trajectory-path.json`,
  reciben su nombre a partir de rutas `sessionFile` de archivos JSONL reales. Los valores `sessionFile` de SQLite son
  marcadores `sqlite:<agentId>:<sessionId>:<storePath>` y no designan archivos
  auxiliares.
- Los lectores del nivel de archivo deben seguir leyendo los archivos JSONL heredados archivados, pero
  la retención del entorno de ejecución no debe explorar los directorios de sesiones activas ni volver a abrir archivos
  de transcripción JSONL para las sesiones de SQLite.

La importación de Doctor sigue siendo la responsable de la migración de los archivos JSONL principales heredados y
sus archivos auxiliares de trayectoria adyacentes. La retención de SQLite en el entorno de ejecución no debe añadir un
segundo importador ni un mecanismo alternativo basado en archivos.

## Puntos de modificación

Ampliar el auxiliar de archivado de SQLite introducido por `clawdbot-d63.1` en lugar de
añadir una ruta paralela.

1. Añadir un recopilador local cerca de `deleteSqliteSessionStateIfUnreferenced`:
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - Incluir `entry.sessionId`, los identificadores de sesión anteriores y posteriores al punto de control, y
     `usageFamilySessionIds`.
   - Filtrar las cadenas vacías y eliminar duplicados de forma determinista.

2. Añadir un recopilador de referencias para el almacén posterior a la eliminación:
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - Recorrer los `session_entries` actuales, analizar cada `entry_json` y recopilar
     los mismos identificadores de familia de cada entrada restante.

3. Cambiar los llamadores de restablecimiento/eliminación/mantenimiento que actualmente archivan un único
   `sessionId` eliminado para que pasen la familia completa de la entrada eliminada.

4. Para cada identificador de familia, archivar las filas de transcripción de SQLite con el
   motivo del llamador (`reset` o `deleted`) y, a continuación, eliminar la fila `sessions` únicamente cuando el
   identificador de familia no esté presente en el conjunto de referencias posterior a la eliminación.

5. Mantener centralizada la eliminación de eventos de transcripción mediante la ruta existente de
   limpieza de filas de sesión de SQLite. No añadir lecturas de archivos JSONL activos.

## Pruebas específicas

Añadir pruebas exclusivas de SQLite a `src/config/sessions/session-accessor.conformance.test.ts`
o a la prueba paralela del ciclo de vida después de que se confirme `clawdbot-d63.1`:

- Al eliminar una entrada con una transcripción anterior a Compaction, se archivan tanto la sesión actual
  como la sesión anterior a Compaction y, a continuación, se eliminan ambos conjuntos de filas de SQLite.
- Al eliminar una de dos entradas que comparten una sesión anterior a Compaction, no se archiva
  nada de la sesión anterior compartida hasta que se elimina la última entrada que hace
  referencia a ella.
- Al eliminar una entrada con `usageFamilySessionIds`, se archivan las filas de transcripción de SQLite
  predecesoras cuando ninguna otra entrada hace referencia a esa familia de uso.
- Una clave de sesión con forma de tema y un marcador de SQLite no provoca ninguna lectura de
  archivos JSONL generados por tema ni ninguna búsqueda de archivos auxiliares.

La comprobación específica debe utilizar:

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

Las comprobaciones generales de `pnpm` deben permanecer en Crabbox/Testbox para este árbol de trabajo de Codex.
