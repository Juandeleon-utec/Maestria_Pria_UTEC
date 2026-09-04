# hardware/

Diseños KiCad, BOM y esquemáticos del sistema de sensado.

**Esta carpeta es candidata a publicarse en el futuro repo open-source** (junto con el firmware, cuando
exista). Mantenerla autocontenida: no referenciar rutas fuera de `hardware/`, y documentar el BOM con
part numbers exactos (ver stack de hardware en el README raíz).

## Contenido esperado (a medida que avance el prototipado)

- `pcb/` — proyecto KiCad (`.kicad_pro`, `.kicad_sch`, `.kicad_pcb`)
- `bom/` — lista de materiales
- `footprints/` — footprint custom del ADXL355 (LCC-14, pendiente de crear — el diseño anterior
  estaba en LGA-14 para ADXL343/345 y no es compatible)
- `gerbers/` — salidas de fabricación (generadas, considerar excluir del historial y regenerar bajo demanda)

## Pendiente crítico

Rediseñar footprint LCC-14 para el ADXL355 antes de fabricar. Ver `referencias/datasheets/adxl354_adxl3551.pdf`.
