# LoRaComms

A GPS-aware, off-grid messaging device built around the ESP32-S3, combining long-range LoRa radio, satellite positioning, and an environmental sensor suite on a custom 4-layer PCB.

LoRaComms is a handheld messenger designed to work with no cellular network, Wi-Fi, or internet connection. It sends text messages over LoRa at 868 MHz (the EU ISM band), tags them with GPS position, and runs entirely from a rechargeable battery — making it suitable for hiking, remote work, emergency communication, and any scenario where conventional infrastructure is unavailable.

The entire device — schematic, 4-layer PCB layout, and RF design — was designed from scratch in EasyEDA.

---

## Features

- **Long-range LoRa messaging** — Semtech SX1262-based radio (Ebyte E22-900M22S, 22 dBm) for kilometre-scale, infrastructure-free communication at 868 MHz.
- **GPS positioning** — on-board ATGM336H GPS/BeiDou receiver with a custom RF front-end (bias-tee) and active antenna, for position tagging and beaconing.
- **Environmental sensing** — barometric altimeter (BMP280) and tilt-compensated digital compass (LSM303AGR) with experimental impact/motion detection.
- **On-device display and input** — 2.42" OLED with a six-button navigation interface for composing and reading messages.
- **Rechargeable power system** — single-cell 18650 Li-ion with on-board charging (TP4056) and an efficient buck-boost regulator (TPS63020) that holds a stable 3.3 V rail across the full battery discharge range.
- **USB-C with native programming** — single-cable power, charging, and firmware flashing via the ESP32-S3's built-in USB controller (no external programmer required), with ESD protection on the data lines.

---

## System Architecture

The ESP32-S3 acts as the central controller, communicating with each peripheral over a dedicated bus:

| Subsystem | Component | Interface |
|---|---|---|
| Long-range radio | Ebyte E22-900M22S (SX1262) | SPI + control lines |
| Satellite positioning | ATGM336H | UART |
| Display | 2.42" SSD1309 OLED | I²C |
| Altimeter / temperature | BMP280 | I²C |
| Compass / accelerometer | LSM303AGR | I²C |
| User input | 6× tactile buttons | GPIO |
| Power | TP4056 charger + TPS63020 buck-boost | — |

The device carries three independent antennas — LoRa (868 MHz), GPS (1.575 GHz), and the ESP32's built-in Wi-Fi/Bluetooth (2.4 GHz) — each on its own frequency and RF path.

---

## Hardware Design

The board is a custom **4-layer PCB** (signal / ground / power / signal) designed for clean power delivery and reliable RF performance:

- **Controlled-impedance RF routing** — 50 Ω microstrip traces for the LoRa and GPS antenna feeds, routed over a solid ground plane with via stitching, with trace widths calculated for the fabricator's stackup.
- **Dedicated ground plane** — a continuous layer-2 ground reference beneath all signal and RF traces, with generous via stitching around the RF sections.
- **Switching-regulator layout** — a tight buck-boost hot-loop (minimised input/output capacitor loop, thermal-via array under the IC) following the regulator's layout guidelines.
- **RF isolation** — the three antennas are physically separated, and noise-sensitive components (the magnetometer, the GPS front-end) are placed away from the switching regulator and high-current paths.
- **Power integrity** — separate copper pours for each supply rail, local decoupling at every IC, and a bias-tee feeding power to the active GPS antenna over its coaxial feed.

---

## Key Components

| Component | Part | Role |
|---|---|---|
| Microcontroller | ESP32-S3-WROOM-1-N8R2 | Main processor, Wi-Fi/BT, native USB |
| LoRa radio | Ebyte E22-900M22S (SX1262) | 868 MHz transceiver |
| GPS | ATGM336H | GNSS receiver |
| Altimeter | BMP280 | Pressure / temperature |
| Compass | LSM303AGR | Magnetometer + accelerometer |
| Charger | TP4056 | Li-ion charging |
| Regulator | TPS63020 | Buck-boost to 3.3 V |
| Battery | 18650 Li-ion (protected) | Power source |
| Display | SSD1309 2.42" OLED | User interface |

---

## Design Highlights

This project involved the full custom-hardware workflow:

- **Schematic capture** of a multi-subsystem embedded device across power, RF, digital, and sensor domains.
- **RF front-end design**, including a bias-tee for the active GPS antenna and controlled-impedance antenna feeds.
- **Power-system design** spanning USB input, Li-ion charging, and buck-boost regulation sized for peak transmit current.
- **4-layer PCB layout** with careful attention to grounding, decoupling, RF isolation, and switching-regulator placement.
- **Design-for-manufacture** — component selection against fabricator stock and assembly capability, and design-rule compliance for the target process.

---

## Status

Hardware design complete; schematic and 4-layer layout finalised for fabrication. Firmware development is the next phase.

### Roadmap
- [x] Schematic design
- [x] 4-layer PCB layout and routing
- [x] RF trace impedance calculation
- [ ] PCB fabrication and assembly
- [ ] Firmware (LoRa messaging, GPS parsing, sensor integration, UI)
- [ ] On-device keyboard input
- [ ] Enclosure design

---

## Tools & Technologies

- **EasyEDA** — schematic capture and PCB layout
- **JLCPCB** — fabrication and assembly
- **ESP-IDF / Arduino** (planned) — firmware
- **RadioLib, TinyGPS++, U8g2** (planned) — peripheral libraries

---

## License

*(Add your chosen license here — e.g. MIT, or "All rights reserved" if you prefer.)*
