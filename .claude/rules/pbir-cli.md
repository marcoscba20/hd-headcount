# Aprendizajes pbir CLI

- `pbir visuals rename` NO acepta el nombre nuevo como argumento posicional: usar `--to "NuevoNombre" -f` (sin `-f` es dry-run y no falla, solo simula).
- Un `pbir set "...**/*.Visual.<container>.<prop>" --remove -f` con glob también toca los visuales de grupo (`visualGroup`) y les deja un stub `"visual": {"objects": {}}` que invalida el schema (`visualType` requerido). Tras un remove masivo, validar y limpiar ese stub en los groupVisual.
- Propiedades `spacing.spaceAboveDivider/spaceAbovePlotArea/spaceAboveSubtitle` las escribe Desktop nuevo pero el schema visualContainer local las rechaza; se pueden quitar en bloque con `pbir set ... --remove -f` sin pérdida (vuelven al default).
- Warnings `SCHEMA_DEGRADED` (visualContainer 2.11.0 no disponible localmente) son informativos: `pbir schema fetch --yes` ya está al día y no los elimina.
- `pbir add visual --title` NO aplica el título cuando se pasa junto con `-d`: el visual queda titulado con el nombre del rol ("Measure"/"Column"). Hay que correr después `pbir visuals title "<path>" --text "..." --show`.
- `pbir rm "<...>.Visual"` borra el `visual.json` pero deja la carpeta vacía. Limpiar con `find . -maxdepth 1 -type d -empty -exec rmdir {} \;` dentro de `visuals/`.
- Los globs (`*.Visual`, `--where`) fallan si el nombre de la página tiene paréntesis, p. ej. `Validación (Turnover).Page`: no matchea nada. Iterar con lista explícita de nombres de visual.
- Textbox: `pbir set "<Visual>.general.paragraphs"` enruta a `visualContainerObjects.general` y rompe el schema. Para textbox con contenido usar `pbir add title` (24pt) o `pbir add subtitle` (14pt gris) — se crean en posición fija con nombre `Title`/`Subtitle`, así que hay que renombrar (`pbir visuals rename --to X -f`) y reposicionar de a uno.
- Las `card` clásicas no envuelven texto: para medidas que devuelven frases hay que fijar `labels.fontSize` bajo (11-15) y `categoryLabels.show=false`.
- Field parameters en TMDL: si la partición calculada es una lista de tuplas anónimas `{ ("X", NAMEOF(...), 0), ... }`, las columnas son `[Value1]`, `[Value2]`, `[Value3]` y NO llevan `isNameInferred`. La columna visible lleva `relatedColumnDetails/groupByColumn` y la de campos `extendedProperty ParameterMetadata = { "version": 3, "kind": 2 }`.
- Nombres de medida con punto interno (`Val · Dif. Conciliación`) confunden al parser `Tabla.Campo` de `-d`. Evitar puntos dentro del nombre.
