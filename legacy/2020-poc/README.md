# 2020 Proof of Concept — Open-Source Predictive Maintenance for AC Induction Motors

This folder contains the 2020 proof-of-concept prototype that precedes the master's thesis work in this repository. It is frozen at release `v0.1-urucon2026`, the version referenced by the paper below.

Low-cost condition monitoring system for AC induction motors based on vibration sensing (ESP8266 + MPU-6050 + LM35), a Raspberry Pi gateway, and LSTM-based degradation forecasting.

This folder accompanies the paper:

> J. P. de León Sum and E. M. Gonçalves, "Open-Source Predictive Maintenance System for AC Induction Motors Using LSTM Neural Networks and Low-Cost Embedded Hardware," in *Proc. IEEE URUCON 2026*.

The original thesis describing this work is available at Zenodo: [10.5281/zenodo.21135685](https://doi.org/10.5281/zenodo.21135685).

## Repository contents

| Folder | Contents |
|---|---|
| `hardware/kicad/` | KiCad schematics and PCB layout of the sensing node |
| `hardware/gerbers/` | Fabrication files (Gerber/drill) as sent to the PCB manufacturer |
| `hardware/enclosure/` | 3D-printable PLA enclosure (STL) |
| `firmware/` | ESP8266 firmware: acquisition, filtering, integration, and transmission |
| `gateway/` | PHP scripts and MariaDB schema running on the Raspberry Pi gateway |
| `ml/` | LSTM training and forecasting code |
| `docs/` | Project documents and their evolution |
| `images/` | Photographs of the prototype and experimental setup |

## Status

This folder contains the materials of the 2020 proof of concept described in the paper. The deployed prototype used a perfboard carrier; the KiCad PCB layout in `hardware/kicad/` is a later revision of the same circuit.

The ongoing master's thesis work (ESP32-S3 nodes, new firmware and models) lives in the rest of this repository.

## System overview

- **Sensing node:** Wemos D1 Mini (ESP8266), MPU-6050 accelerometer on a GY-521 breakout, LM35 temperature sensor.
- **Acquisition:** 1 kHz sampling, 0.5 s blocks (N = 500), 10 Hz high-pass filtering, trapezoidal integration to vibration velocity, one record per second.
- **Gateway:** Wi-Fi router board (access point and DHCP) and Raspberry Pi 3 Model B with Apache, PHP, and MariaDB.
- **Model:** single-layer LSTM (32 units) forecasting X-axis acceleration variance.

## License

See the `LICENSE` file at the root of this repository.

## Citation

If you use this work, please cite the paper above (see `CITATION.cff`).
