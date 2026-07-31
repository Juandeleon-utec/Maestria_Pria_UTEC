# Framework Open-Source de Monitoreo Predictivo Industrial mediante Edge AI

Trabajo Final de Maestría en Robótica e Inteligencia Artificial (PRIA) — UTEC, Uruguay
**Autor:** Juan Pedro de León Sum

## Descripción

Sistema distribuido de código abierto para el monitoreo predictivo (CBM) de motores de inducción trifásicos, validado en una planta agroindustrial uruguaya con más de 100 motores en operación continua. El proyecto propone una arquitectura jerárquica de IA en tres niveles:

- **TCN** (Temporal Convolutional Networks) cuantizadas en nodos embebidos ESP32-S3, para detección de anomalías en tiempo real a partir de vibración (ADXL355) y temperatura (DS18B20).
- **TFT** (Temporal Fusion Transformers) en servidor centralizado, para pronóstico multivariable de Vida Útil Remanente (RUL) con horizonte de 24 horas.
- **GNN** (Graph Neural Networks), para modelar la propagación de vibraciones entre motores físicamente adyacentes y reducir falsos positivos por contaminación cruzada.

El trabajo es una continuación directa del Trabajo Final de Especialización del autor (2021, arquitectura LSTM, TRL 4), y busca elevar el sistema a **TRL 7** mediante despliegue y validación en entorno operativo real, bajo la normativa ISO 20816-1:2016.

## Estado actual

🚧 En desarrollo — Capítulos de Introducción, Objetivos y Estado del Arte redactados. Diseño de hardware (PCB, KiCad) en curso.

## Estructura del repositorio

- `main.tex` / `Biblio.bib` — documento de tesis (LaTeX)
- `hardware/` — diseño PCB (KiCad), BOM
- `firmware/` — ESP-IDF + FreeRTOS (ESP32-S3)
- `models/` — TCN, TFT, GNN (PyTorch)

## Licencia

Este proyecto es de código abierto bajo licencia [MIT](https://opensource.org/licenses/MIT).
