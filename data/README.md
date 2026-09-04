# data/

- `processed/` — tablas de datos curadas, resúmenes, features extraídas, exports pequeños (CSV/Parquet
  livianos). Esto sí va en git normal.
- `raw/` — **no versionar acá** datos crudos de sensores (series temporales de vibración/temperatura de
  los nodos). Cuando empiece la captura real en planta, esos volúmenes van a InfluxDB/backups aparte, no
  al historial de git. Carpeta agregada al .gitignore por defecto.

Papers y datasheets siguen en git normal (referencias/) — bajo cambio (~1/semana), no justifica Git LFS.
