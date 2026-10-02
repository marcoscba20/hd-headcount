# Parche temporal: Cross Charge Entity Name de personas Global

**Estado:** ACTIVO desde 2026-09-23 · **Retirar cuando** Oracle traiga el Cross Charge de estas personas.

## Motivo
Las personas Global (contratadas por 270 HD Brazil, pero que trabajan para entidades cross-charge) figuran en el export de Oracle sin `PER_ASG_DF_CRS_CHRG_NTY_N_v`. En el modelo quedan como "Sin Datos" y no suman en `[Dotación Cross-Entity]`. HR Brasil informó la zona de cada una en un archivo manual.

## Fuente
`OneDrive - hdlaoti\People LATAM - Documentos\Área de Trabajo - Marcos\Modelos de Datos Power BI\HeadCount Oracle\Global Employees - Fill by local HR_Brazil AGO.xlsx`

- En la hoja `2026` están los datos de HR. No se modificó.
- La hoja `Parche PBI` es auxiliar y se agregó para este parche. Tiene fórmulas que leen `2026`:
  - `Person Number` = Oracle ID. Las filas sin Oracle ID no cruzan (14 personas).
  - `Zona Parche` = `UPPER(TRIM(Zone))`. Si Zone está vacía o dice `NA`, se toma del mapeo Entity→Zona (G2:H9): 211 GLOBAL, 210/213 HDNA, 118 GBP, 293 S BLINDS US, 620 EMEA, 165 WFASHION.

## Lógica en el modelo
- Expresión `Parche Cross Charge Global` (`expressions.tmdl`): lee `Parche PBI`. Si el archivo o la hoja no existen, devuelve una tabla vacía y el refresh no falla.
- Partición `Excel HC Oracle`, pasos `PersonasConCC` … `#"Cross Charge Parcheado"`:
  - Se completa `Cross Charge Entity Name` con la **zona** solo si la persona no tiene Cross Charge en **ninguna** fila de Oracle. Nunca se pisa un valor que venga de Oracle.
  - Se completan todas las filas de esa persona, incluidas las INACTIVE_PROCESS.
- La columna nueva `Cross Charge Origen` toma los valores `Oracle` / `Parche manual (Global Employees)` / `Sin Datos`.

Nota: en la columna conviven nombres de entidad de Oracle ("211 HD Holdings") y zonas del parche ("GLOBAL"). Es una decisión deliberada.

La columna `Ámbito` (Global / LATAM / Local) se calcula a partir de `Cross Charge Entity Name` ya parcheado. La zona `LATAM` del parche da LATAM y las demás zonas dan Global. Si se retira el parche, `Ámbito` sigue funcionando, pero esas 21 personas pasan a Local hasta que Oracle cargue su Cross Charge.

## Personas completadas al 2026-09-23 (21 personas, 39 filas)
| Person Number | Zona |
|---|---|
| 52000019 | LATAM |
| 52000782 | GLOBAL |
| 52005400 | S BLINDS US |
| 52005415 | HDNA |
| 52007303 | EMEA |
| 52007395 | S BLINDS US |
| 52007640 | S BLINDS US |
| 53023228 | GBP |
| 53024266 | GBP |
| 53024272 | GLOBAL |
| 53024274 | S BLINDS US |
| 53024316 | HDNA |
| 53024322 | GLOBAL |
| 53024330 | HDNA |
| 53027902 | HDNA (Zone "NA", inferida por Entity 213) |
| 53028053 | WFASHION |
| 53028084 | GLOBAL |
| 53028085 | GLOBAL |
| 53028088 | GLOBAL |
| 53028166 | GLOBAL |
| 53028188 | HDNA |

## Seguimiento
Para ver qué personas siguen dependiendo del parche, filtrar `Excel HC Oracle[Cross Charge Origen] = "Parche manual (Global Employees)"`. Cuando Oracle carga el dato de una persona, esa persona deja de ser parcheada sola y pasa a `Oracle`.

## Cómo retirarlo
Cuando el filtro anterior devuelva 0 personas:
1. En la partición `Excel HC Oracle`, borrar los pasos entre `// --- PARCHE TEMPORAL` y `#"Cross Charge Parcheado"`. Hacer que `#"Grade Nº Agregado"` vuelva a partir de `#"Blindaje Vacios Textos"`.
2. Borrar la columna `Cross Charge Origen` de la TMDL, junto con los visuales que la usen si los hubiera. Borrar también la expresión `Parche Cross Charge Global` y su entrada en `PBI_QueryOrder` (`model.tmdl`).
3. Quitar la nota de parche de la descripción de `Cross Charge Entity Name`.
4. Marcar este documento como RETIRADO. La hoja `Parche PBI` del Excel se puede borrar.
