# Framework Open-Source Distribuido para Monitoreo Predictivo Industrial de Motores de Inducción mediante Edge AI y Arquitecturas de Atención Temporal

Trabajo Final de Maestría (TFM) — Programa de Robótica e Inteligencia Artificial (PRIA), Universidad Tecnológica (UTEC), Uruguay.

Autor: Juan Pedro de León Sum
Continuación del Trabajo Final de Especialización (TFE, 2021) — LSTM, 95.30% accuracy, TRL 4.
Objetivo del TFM: TRL 7, 10 nodos sensores en planta agroindustrial real (Uruguay), 100+ motores trifásicos de inducción.

## Estructura del repositorio

```
.
├── latex/                 # Fuente LaTeX de la tesis (main.tex, Biblio.bib)
├── docs/
│   ├── propuesta/          # Versiones de la propuesta de TFM (docx/pdf)
│   └── figuras/             # Infografías y diagramas propios
├── hardware/               # Diseños KiCad, BOM, esquemáticos (pendiente de agregar)
└── referencias/
    ├── papers/              # Artículos de terceros usados en el estado del arte
    └── datasheets/          # Datasheets de componentes (ESP32-S3, ADXL355, ADS1115, DS18B20)
```

## Stack de hardware (definitivo)

| Componente | Detalle |
|---|---|
| MCU | ESP32-S3 |
| Acelerómetro | ADXL355 (SPI, LCC-14, 6×5.6×2.2 mm) |
| Temperatura | DS18B20 (1-Wire) |
| Regulador LDO | TLV75733PDBVR (3.3V) |
| Conectores | JST-XH 2-pin |
| Diseño PCB | KiCad |

> Pendiente: rediseño de footprint LCC-14 para ADXL355 (difiere del LGA-14 usado previamente para ADXL343/345).

## Arquitectura de IA (tres niveles)

1. **Edge (ESP32-S3):** TCN cuantizado para inferencia local
2. **Servidor central:** Temporal Fusion Transformers (TFT) para prognosis RUL a 24h
3. **Fleet-level:** Graph Neural Networks (GNN) para modelado de acoplamiento vibracional inter-motor

## Publicaciones

- IEEE URUCON 2026 — paper ID 1571307323
- Zenodo preprint — DOI [10.5281/zenodo.19008609](https://doi.org/10.5281/zenodo.19008609)

## Nota sobre `referencias/`

Los PDFs en `referencias/papers/` y `referencias/datasheets/` son material de terceros (papers académicos y datasheets de fabricantes) incluidos como referencia de trabajo interno. **Si este repositorio se publica en un GitHub público, revisar la licencia de cada editorial/fabricante antes de mantenerlos en el historial** — muchas editoriales (IEEE, Elsevier, MDPI según el caso) no permiten redistribución del PDF final. Considerar mantener esta carpeta en un repo privado o usar `.gitignore` + un `referencias/README.md` con los DOIs en vez de los archivos, si se planea hacer público.

## Plan de repositorios: este repo (privado) vs. futuro repo público

**Este repositorio es privado** y contiene todo el trabajo de investigación de la maestría: propuesta,
papers de terceros, datasheets, avances de PCB en KiCad, notas y borradores de tesis. Se usa como
único punto de sincronización entre PC de casa, PC de UTEC y notebook.

**A futuro**, al terminar la maestría, se planea abrir un **repositorio público separado** (para
postular a fondo de inversión / publicar como open-source) que contendrá **únicamente**:
- Código fuente (firmware ESP32-S3, modelos, pipelines de entrenamiento)
- Esquemáticos y PCB de KiCad (`hardware/`)
- La publicación propia final (paper, no los papers de terceros)

**No pasará al repo público:** `referencias/` (papers de terceros, datasheets con copyright de
fabricante), `docs/propuesta/` (borradores administrativos), ni notas internas de tesis.

Para que esa separación sea fácil el día que corresponda, mantené la disciplina de:
- Todo el código y diseño hardware "publicable" vive limpio en `hardware/` y (cuando exista) `firmware/`
  o `src/`, sin mezclarlo con archivos de investigación.
- Cuando llegue el momento, se puede extraer solo esas carpetas al nuevo repo con `git subtree split`
  o simplemente copiando el estado final (no hace falta arrastrar el historial de investigación).

## Flujo de trabajo multi-máquina (casa / UTEC / notebook)

Como es privado, el repo debe vivir en GitHub (u otro host privado) y sincronizarse por git normal:

```bash
# En cada máquina nueva (una sola vez):
git clone git@github.com:<usuario>/<repo-privado>.git
cd <repo-privado>

# Rutina de trabajo en cualquier máquina:
git pull                      # traer lo último antes de empezar
# ... trabajar en KiCad / LaTeX / lo que sea ...
git add -A
git commit -m "mensaje descriptivo"
git push
```

Puntos a cuidar:
- **Antes de abrir KiCad en una máquina, siempre `git pull` primero** — los archivos `.kicad_pcb`/`.kicad_sch`
  son texto plano pero los merges automáticos de edición simultánea son propensos a conflictos; evitá editar
  el mismo proyecto PCB desde dos máquinas sin sincronizar entre medio.
- Configurá la misma clave SSH (o credential helper) en las tres máquinas para no reescribir `user.email`/`user.name` cada vez.
- Si en algún momento el repo pesa demasiado (los papers ya suman ~190 MB), considerá `git lfs` para los PDFs
  y binarios grandes, para que los clones/pulls en la notebook (posiblemente con mala conexión) sean más rápidos.

## Licencia

MIT License — ver `LICENSE`. Aplica al código, diseños de hardware y texto propio del autor. No aplica a los materiales de terceros en `referencias/`.
