---
x-i18n:
    generated_at: "2026-07-26T04:29:51Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# Guía de documentación

Este directorio abarca la redacción de documentación, las reglas de enlaces de Mintlify y la política de internacionalización de la documentación.

## Reglas de Mintlify

- La documentación se aloja en Mintlify (`https://docs.openclaw.ai`).
- Los enlaces internos de la documentación en `docs/**/*.md` deben permanecer relativos a la raíz y no incluir el sufijo `.md` ni `.mdx` (ejemplo: `[Config](/gateway/configuration)`).
- Las referencias cruzadas entre secciones deben usar anclas en rutas relativas a la raíz (ejemplo: `[Hooks](/gateway/configuration-reference#hooks)`).
- Los encabezados de la documentación deben evitar las rayas y los apóstrofos, ya que la generación de anclas de Mintlify es frágil en esos casos.
- El README y otros documentos renderizados en GitHub deben conservar las URL absolutas de la documentación para que los enlaces funcionen fuera de Mintlify.
- El contenido de la documentación debe ser genérico: no debe incluir nombres de dispositivos personales, nombres de host ni rutas locales; deben usarse marcadores de posición como `user@gateway-host`.

## Reglas de contenido de la documentación

- En la documentación, los textos de la interfaz de usuario y las listas de selectores, los servicios y proveedores deben ordenarse alfabéticamente, salvo que la sección describa explícitamente el orden de ejecución o el orden de detección automática.
- La nomenclatura de los plugins incluidos debe ser coherente con las reglas terminológicas para plugins aplicables a todo el repositorio que se encuentran en el archivo `AGENTS.md` raíz.
- Documentación generada que nunca debe editarse manualmente: `docs/plugins/reference/**`, `docs/plugins/reference.md` y `docs/plugins/plugin-inventory.md` se generan a partir de `pnpm plugins:inventory:gen`; `docs/docs_map.md`, a partir de `pnpm docs:map:gen`; `docs/maturity/**`, a partir de `pnpm maturity:render`.

## Documentación interna

- La documentación privada de larga duración para operadores debe almacenarse en `~/Projects/manager/docs/`.
- La documentación interna temporal o duplicada, local al repositorio, puede almacenarse en el directorio ignorado `docs/internal/`.
- Nunca se deben añadir páginas de `docs/internal/**` a la navegación de `docs/docs.json` ni enlazarlas desde la documentación pública.
- `scripts/docs-sync-publish.mjs` excluye y elimina `docs/internal/**` del repositorio público de publicación `openclaw/docs` si posteriormente se fuerza la adición de una página.
- La documentación interna puede mencionar rutas del repositorio, nombres de aplicaciones privadas, nombres de elementos de 1Password y manuales operativos, pero nunca debe incluir valores secretos.

## Edición de la tabla de evaluación de madurez

`taxonomy.yaml` y `qa/maturity-scores.yaml` son las entradas de origen; la documentación de madurez generada en `docs/maturity/` es una proyección y no debe editarse manualmente para modificar la puntuación, el LTS, la taxonomía, el perfil de QA ni las tablas de evidencias.
`scripts/qa/render-maturity-docs.ts` gestiona la generación; se debe usar `pnpm maturity:render` para actualizar la documentación confirmada en el repositorio y `pnpm maturity:check` para verificarla.
`.github/workflows/maturity-scorecard.yml` renderiza vistas previas de los artefactos y puede abrir pull requests de documentación generada; `.github/workflows/openclaw-release-checks.yml` lo ejecuta para el QA de las versiones.
Los datos deterministas de `qa-evidence.json.scorecard` deben conservarse en los artefactos de GitHub Actions, salvo que un responsable solicite explícitamente una proyección depurada y confirmada en el repositorio.
Las modificaciones manuales deben cambiar el estado de origen en un pull request y explicar el motivo, además de aportar evidencias públicas o censuradas.

## Internacionalización de la documentación

- La documentación en otros idiomas no se mantiene en este repositorio. El resultado de publicación generado se encuentra en el repositorio independiente `openclaw/docs` (que suele clonarse localmente como `../openclaw-docs`).
- No se debe añadir ni editar aquí documentación localizada en `docs/<locale>/**`.
- La documentación en inglés de este repositorio y los archivos de glosario deben considerarse la fuente de verdad.
- Pipeline: actualizar aquí la documentación en inglés, actualizar `docs/.i18n/glossary.<locale>.json` según sea necesario y, a continuación, permitir que se ejecuten la sincronización del repositorio de publicación y `scripts/docs-i18n` en `openclaw/docs`.
- Antes de volver a ejecutar `scripts/docs-i18n`, deben añadirse entradas al glosario para todos los términos técnicos, títulos de páginas o etiquetas de navegación breves nuevos que deban permanecer en inglés o usar una traducción fija.
- `pnpm docs:check-i18n-glossary` es la comprobación para los títulos modificados de la documentación en inglés y las etiquetas internas breves de la documentación.
- La memoria de traducción se encuentra en los archivos `docs/.i18n/*.tm.jsonl` generados en el repositorio de publicación.
- Consulte `docs/.i18n/README.md`.
