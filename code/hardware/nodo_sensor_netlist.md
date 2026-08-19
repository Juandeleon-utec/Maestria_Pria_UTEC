# Nodo Sensor ESP32-S3 + ADXL355 (SPI) + DS18B20 (1-Wire)
## Netlist y notas de diseño

Fuentes: `esp32s3wroom1_wroom1u_datasheet_en.pdf`, `adxl354_adxl3551.pdf`, `DS18B20.pdf` (proyecto), + datasheet TLV75733PDBVR (TI, verificado por búsqueda).

**Supuestos confirmados:**
- Conector de entrada 5V: **header JST-XH 2 pines**.
- DS18B20: **alimentación externa** (VDD → 3.3V), montado en la carcasa del motor mediante cable externo de **hasta 20 cm** — a esa longitud la caída de tensión y la capacitancia parásita del cable son despreciables para el consumo del sensor (~1,5 mA en modo activo), así que no se requieren ajustes de calibre de cable ni de diseño por la distancia.

---

## ⚠️ Hallazgo crítico antes del netlist

Tu tabla de riesgos ya menciona PSRAM de 8MB para el buffer MQTT. Eso implica que estás usando la variante **ESP32-S3R8 / WROOM con PSRAM Octal**, en la cual **IO35, IO36 e IO37 quedan reservados internamente para la PSRAM y no están disponibles como GPIO de propósito general** (confirmado en el datasheet del módulo, nota "b" de la tabla de pines). Si en algún diseño previo asignaste alguno de esos tres pines a SPI o al 1-Wire, hay que reasignarlo — no vas a poder usarlos.

También evité los **pines de strapping** (GPIO0, GPIO3, GPIO45, GPIO46), que definen el modo de arranque del chip y no deben cargarse con circuitería externa que fuerce un nivel lógico distinto al esperado en el reset.

---

## 1. Dominios de alimentación

```
5V (JST-XH, 2 pines) ──┬── TLV75733PDBVR (IN, pin 2)
                        │
                       GND
                        
TLV75733PDBVR (OUT, pin 5) ── 3V3_RAIL
  - CIN  = 1 µF cerámico, entre IN (pin 2) y GND (pin 1), lo más cerca posible del regulador
  - COUT = 1 µF cerámico, entre OUT (pin 5) y GND (pin 1)
  - EN (pin 3) → 3V3_RAIL directo (habilitado permanente; no hay modo shutdown por software en esta versión)
  - Pin 4 (NC) → sin conexión
```

El ADXL355 tiene **dos dominios de alimentación distintos que no deben unirse a la ligera**: VSUPPLY (analógico, pin 11) y VDDIO (interfaz digital, pin 5). En este diseño de un solo riel de 3,3V, ambos se alimentan del mismo 3V3_RAIL, pero **cada uno lleva su propio capacitor de desacoplo en su propio pin** — no compartir un solo capacitor entre los dos.

---

## 2. Bus SPI — ESP32-S3 ↔ ADXL355

Uso el periférico **FSPI** (SPI2) del módulo, evitando los pines de PSRAM y de strapping.

| Señal SPI | Pin ESP32-S3-WROOM-1 (GPIO) | Pin ADXL355 (N°, nombre) |
|---|---|---|
| SCLK | GPIO12 (FSPICLK) | Pin 2, SCLK/VSSIO |
| MOSI | GPIO11 (FSPID) | Pin 3, MOSI/SDA |
| MISO | GPIO13 (FSPIQ) | Pin 4, MISO/ASEL |
| CS   | GPIO10 (FSPICS0) | Pin 1, CS/SCL |

**Nota de configuración crítica:** el pin 2 del ADXL355 es multifunción (SCLK en modo SPI, o VSSIO — es decir, debe ir a GND — para habilitar modo I²C). Como vamos por SPI, este pin **debe quedar conectado al SCLK del ESP32, nunca a GND**, o el sensor entra en modo I²C y el bus deja de funcionar. Timing: SPI Mode 0 (CPOL=0, CPHA=0), velocidad de reloj entre 100 kHz y 10 MHz (el datasheet no recomienda operar fuera de ese rango).

---

## 3. Bus 1-Wire — ESP32-S3 ↔ DS18B20

| Señal | Pin ESP32-S3-WROOM-1 (GPIO) | Pin DS18B20 (TO-92) |
|---|---|---|
| DQ (datos) | GPIO4 | Pin 2, DQ (vía cable externo ≤20 cm + conector de 3 pines en la placa) |
| VDD | 3V3_RAIL | Pin 3, VDD (ídem) |
| GND | GND | Pin 1, GND (ídem) |

- Resistencia de pull-up: **4,7 kΩ entre DQ y 3V3_RAIL** (según tu especificación y coincide con el valor que usa el propio fabricante en el circuito de referencia del datasheet).
- Con alimentación externa (no parasite power), no se requiere el MOSFET de pull-up fuerte que exige el modo parasite — el circuito es más simple.

---

## 4. Desacoplo por integrado

| Integrado | Pin de alimentación | Capacitor | Ubicación |
|---|---|---|---|
| ESP32-S3-WROOM-1 | 3V3 (pin 2) | 100 nF + 10 µF en paralelo | Lo más cerca posible del pin 2 del módulo |
| ADXL355 | VSUPPLY (pin 11) | 100 nF + 10 µF en paralelo | Junto al pin 11 |
| ADXL355 | VDDIO (pin 5) | 100 nF | Junto al pin 5 |
| ADXL355 | V1P8DIG (pin 8) | 100 nF | Junto al pin 8 (requerido por datasheet, aunque sea salida del LDO interno) |
| ADXL355 | V1P8ANA (pin 10) | 100 nF | Junto al pin 10 (ídem) |
| DS18B20 | VDD (pin 3) | 100 nF | Junto al sensor (recomendado para el modo de alimentación externa, no es estrictamente obligatorio en el datasheet pero es buena práctica) |
| TLV75733 | IN (pin 2) / OUT (pin 5) | 1 µF cerámico c/u | Ver sección 1 |

Los pines 8 y 10 del ADXL355 (V1P8DIG, V1P8ANA) son salidas del regulador LDO *interno* del sensor cuando VSUPPLY está entre 2,25V y 3,6V — no se conectan a ningún riel externo, pero el datasheet exige que cada uno lleve su propio capacitor de desacoplo a GND igual.

---

## 5. Netlist completo pin a pin

| # | Net | Origen | Destino |
|---|---|---|---|
| 1 | 5V_IN | JST-XH pin 1 | TLV75733 pin 2 (IN) |
| 2 | GND | JST-XH pin 2 | Plano de tierra común (todos los GND) |
| 3 | 3V3_RAIL | TLV75733 pin 5 (OUT) | ESP32-S3 pin 2 (3V3) |
| 4 | 3V3_RAIL | TLV75733 pin 5 (OUT) | ADXL355 pin 11 (VSUPPLY) |
| 5 | 3V3_RAIL | TLV75733 pin 5 (OUT) | ADXL355 pin 5 (VDDIO) |
| 6 | 3V3_RAIL | TLV75733 pin 5 (OUT) | DS18B20 pin 3 (VDD) |
| 7 | 3V3_RAIL | TLV75733 pin 5 (OUT) | TLV75733 pin 3 (EN) |
| 8 | SPI_SCLK | ESP32-S3 GPIO12 | ADXL355 pin 2 (SCLK/VSSIO) |
| 9 | SPI_MOSI | ESP32-S3 GPIO11 | ADXL355 pin 3 (MOSI/SDA) |
| 10 | SPI_MISO | ESP32-S3 GPIO13 | ADXL355 pin 4 (MISO/ASEL) |
| 11 | SPI_CS | ESP32-S3 GPIO10 | ADXL355 pin 1 (CS/SCL) |
| 12 | ONEWIRE_DQ | ESP32-S3 GPIO4 | DS18B20 pin 2 (DQ) |
| 13 | ONEWIRE_DQ | DS18B20 pin 2 (DQ) | R_pullup (4,7 kΩ) → 3V3_RAIL |
| 14 | GND | ESP32-S3 pin 1 (GND) | Plano de tierra común |
| 15 | GND | ADXL355 pin 9 (VSS, analógico) | Plano de tierra común |
| 16 | GND | ADXL355 pin 6 (VSSIO, digital) | Plano de tierra común |
| 17 | GND | DS18B20 pin 1 (GND) | Plano de tierra común |
| 18 | GND | TLV75733 pin 1 (GND) | Plano de tierra común |
| 19 | RESERVED | ADXL355 pin 7 | GND o sin conectar (según datasheet) |
| 20 | NC | TLV75733 pin 4 | Sin conexión |

**Nota sobre GND único:** el datasheet del ADXL355 separa VSS (analógico) y VSSIO (digital) como dominios distintos, pero para esta aplicación de una sola tarjeta con un plano de tierra continuo, ambos se unen al mismo plano — la separación analógico/digital importa más cuando hay ruido de conmutación digital cercano (como un regulador switching), lo cual no es el caso acá con un LDO lineal. Si en el futuro cambiás a un regulador switching (buck), ahí sí conviene mantener las tierras separadas con una única unión en un punto (estrella).

---

## 6. Pendiente de tu parte antes de fabricar

- Verificar footprint LCC-14 del ADXL355 en KiCad (ya lo tenés como pendiente desde la migración de sensor).
- Definir el conector de 3 pines en placa para el cable remoto del DS18B20 (JST-XH o similar, a definir tamaño según el cable que uses).
- Este documento no incluye el filtrado EMI adicional (planos de masa, apantallamiento) que mencionás en tu tabla de riesgos — es un tema de layout de PCB, no de netlist, y depende de cómo distribuyas los componentes físicamente.
