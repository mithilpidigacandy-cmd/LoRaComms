# LoRaComms

A GPS-aware, off-grid messaging device built around the ESP32-S3, combining long-range LoRa radio, satellite positioning, and an environmental sensor suite on a custom 4-layer PCB.

LoRaComms is a handheld messenger designed to work with no cellular network, Wi-Fi, or internet connection. It sends text messages over LoRa at 868 MHz (the EU ISM band), tags them with GPS position, and runs entirely from a rechargeable battery — making it suitable for hiking, remote work, emergency communication, and any scenario where conventional infrastructure is unavailable.

The entire device — schematic, 4-layer PCB layout, and RF design — was designed from scratch in EasyEDA.

---

## Features

- **Long-range LoRa messaging** — Semtech SX1262-based radio (Ebyte E22-900M22S, 22 dBm) for kilometre-scale, infrastructure-free communication at 868 MHz.
- **GPS positioning** — on-board ATGM336H GPS/BeiDou receiver with a custom RF front-end (bias-tee) and active antenna, for position tagging and beaconing.
- **Environmental sensing** — barometric altimeter and temperature/humidity sensor (BME280) plus a tilt-compensated digital compass (LSM303AGR) with experimental impact/motion detection.
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
| Pressure / temp / humidity | BME280 | I²C |
| Compass / accelerometer | LSM303AGR | I²C |
| User input | 6× tactile buttons | GPIO |
| Power | TP4056 charger + TPS63020 buck-boost | — |

The device carries three independent antennas — LoRa (868 MHz), GPS (1.575 GHz), and the ESP32's built-in Wi-Fi/Bluetooth (2.4 GHz) — each on its own frequency and RF path.

The display, pressure sensor, and compass all share a single I²C bus (four devices at distinct addresses, one pull-up pair), while the LoRa radio sits on its own SPI bus with dedicated RF-switch control lines.

---

## Hardware Design

The board is a custom **4-layer PCB** (signal / ground / power / signal) designed for clean power delivery and reliable RF performance:

- **Controlled-impedance RF routing** — 50 Ω microstrip traces for the LoRa and GPS antenna feeds, routed over a solid ground plane with via stitching. The trace widths were calculated specifically for the chosen JLCPCB stackup (see *RF Impedance Design* below).
- **Dedicated ground plane** — a continuous layer-2 ground reference beneath all signal and RF traces, with generous via stitching around the RF sections.
- **Switching-regulator layout** — a tight buck-boost hot-loop (minimised input/output capacitor loop, thermal-via array under the IC) following the regulator's layout guidelines, with the feedback trace routed clear of the switching node.
- **RF isolation** — the three antennas are physically separated, and noise-sensitive components (the magnetometer, the GPS front-end) are placed away from the switching regulator and high-current paths.
- **Power integrity** — separate copper pours for each supply rail (3.3 V distribution, local battery and 5 V zones), local decoupling at every IC, and a bias-tee feeding power to the active GPS antenna over its coaxial feed.

### RF Impedance Design

Getting the two antenna feeds to a true 50 Ω was the most delicate part of the layout, and it was handled end-to-end:

- **Stackup-specific calculation** — the LoRa and GPS RF traces were not left to a guessed width. Each candidate JLCPCB 4-layer stackup specifies a different dielectric height between the top signal layer and the layer-2 ground plane, which changes the width needed for 50 Ω. The trace width was calculated from the actual dielectric thickness and permittivity of the ordered stackup rather than assumed.
- **Stackup selection** — JLCPCB's standard 4-layer stackup (`JLC04161H-7628`, a single 0.2104 mm 7628 prepreg between L1 and L2) was selected, giving a **50 Ω microstrip width of ≈ 0.364 mm**. This standard stackup was chosen deliberately over a "special" alternative that would have carried a higher fabrication cost for a negligible difference in trace width.
- **Verification** — the final RF traces were confirmed against the target width, kept short, routed entirely over continuous ground, and flanked with stitched ground pour. Because both RF runs are short (each radio sits immediately beside its antenna connector), the design is tolerant of the small manufacturing variation inherent in a standard (non-impedance-controlled) stackup.

This process — understanding that impedance depends on the fabricator's layer stackup, reading the real dielectric parameters, calculating the matching trace width, and balancing RF precision against cost — reflects the kind of trade-off decision that distinguishes a manufacturable design from a textbook one.

---

## Key Components

| Component | Part | Role |
|---|---|---|
| Microcontroller | ESP32-S3-WROOM-1-N8R2 | Main processor, Wi-Fi/BT, native USB |
| LoRa radio | Ebyte E22-900M22S (SX1262) | 868 MHz transceiver |
| GPS | ATGM336H | GNSS receiver |
| Pressure / temp / humidity | BME280 | Environmental sensing |
| Compass | LSM303AGR | Magnetometer + accelerometer |
| Charger | TP4056 | Li-ion charging |
| Regulator | TPS63020 | Buck-boost to 3.3 V |
| Battery | 18650 Li-ion (protected) | Power source |
| Display | SSD1309 2.42" OLED | User interface |
| Antenna connectors | SMA (LoRa), u.FL (GPS) | External antenna feeds |

---

## Design Highlights

This project involved the full custom-hardware workflow:

- **Schematic capture** of a multi-subsystem embedded device across power, RF, digital, and sensor domains, built up and verified block by block.
- **RF front-end design**, including a bias-tee for the active GPS antenna and controlled-impedance antenna feeds.
- **Power-system design** spanning USB input, Li-ion charging, and buck-boost regulation sized for peak transmit current.
- **4-layer PCB layout** with careful attention to grounding, decoupling, RF isolation, and switching-regulator placement.
- **RF impedance engineering** — calculating 50 Ω trace widths from the fabricator's real stackup parameters and selecting the stackup to balance performance against cost.
- **Design-for-manufacture** — component selection against live fabricator stock and assembly capability (including sourcing substitutions when a part became unavailable), and design-rule compliance for the target process.

---

## Status

Hardware design complete; schematic and 4-layer layout finalised for fabrication, with impedance-matched RF traces and a manufacturing-ready stackup selection. Firmware development is the next phase.

### Roadmap
- [x] Schematic design
- [x] 4-layer PCB layout and routing
- [x] RF trace impedance calculation and stackup selection
- [x] Design-rule check and manufacturing prep
- [ ] PCB fabrication and assembly
- [ ] Firmware (LoRa messaging, GPS parsing, sensor integration, UI)
- [ ] On-device text entry (scrolling character selector / canned messages)
- [ ] Enclosure design

---

## Tools & Technologies

- **EasyEDA** — schematic capture and PCB layout
- **JLCPCB** — fabrication and assembly (4-layer, `JLC04161H-7628` standard stackup)
- **ESP-IDF / Arduino** (planned) — firmware
- **RadioLib, TinyGPS++, U8g2, Adafruit BME280 / LSM303** (planned) — peripheral libraries

---

## License

*(Add your chosen license here — e.g. MIT, or "All rights reserved" if you prefer.)*
