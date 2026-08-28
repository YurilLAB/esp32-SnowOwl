<div align="center">

# 🦉 ESP32-SnowOwl

**A multi-purpose wireless _offensive &amp; defensive_ security multitool.**

Three processors — **ESP32-S3 + ESP32-C5 (5 GHz Wi‑Fi) + ARM Linux** — driving a wall of radios and readers: dual‑band Wi‑Fi, BLE, 2.4 GHz, Sub‑GHz, **LoRa**, IR, **NFC**, **125 kHz RFID**, **iButton** &amp; GPS, in one stackable handheld.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Compute](https://img.shields.io/badge/compute-ESP32--S3%20%2B%20ESP32--C5%20%2B%20ARM%20Linux-blue)
![Radios](https://img.shields.io/badge/radios-WiFi%202.4%2F5GHz%20%C2%B7%20BLE%20%C2%B7%20SubGHz%20%C2%B7%20LoRa%20%C2%B7%20nRF24-purple)
![PCB](https://img.shields.io/badge/PCB-KiCad%2010%20%C2%B7%204--layer-orange)
![Status](https://img.shields.io/badge/status-in%20development-yellow)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)

</div>

> ⚠️ **Authorized use only.** SnowOwl is built for security research, education, CTFs, and testing on systems you own or are explicitly authorized to assess. You are solely responsible for complying with all applicable laws and regulations.

---

## 📖 About

**ESP32-SnowOwl** is a hardware evolution of [CiferTech's **ESP32‑DIV**](https://github.com/cifertech/ESP32-DIV) — the open-source ESP32‑S3 wireless multitool. SnowOwl keeps the entire ESP32‑DIV toolset and then piles on the compute, radios and readers the stock board doesn't have.

**Three processors, each doing what it's best at:**

- 🦅 **ESP32‑S3 — the controller.** The base ESP32‑DIV brain: the on‑device UI, 2.4 GHz Wi‑Fi &amp; BLE, and orchestration of every radio.
- 📡 **ESP32‑C5 — the 5 GHz reach.** A dual‑band **Wi‑Fi 6 (2.4 + 5 GHz)** co‑processor that adds the entire **5 GHz** band the S3 can't touch, on its own antenna and its own regulated rail.
- 🐧 **RV1106 ARM Linux — the analyst.** A real Linux computer (Luckfox Core1106) for capture, monitoring, scripting and heavy on‑device tooling — self‑powered from **its own USB‑C**.

Offense on the talons, analysis in the eyes. Silent, patient, built for the dark — the snow owl fits.

---

## 🦉 What SnowOwl adds over ESP32‑DIV

| Addition | Part | What it brings |
|---|---|---|
| 📡 **5 GHz Wi‑Fi co‑processor** | ESP32‑C5‑WROOM‑1U | Dual‑band **Wi‑Fi 6 (2.4 + 5 GHz)** — the 5 GHz band the ESP32‑S3 can't reach. Own U.FL antenna, own AP7361C LDO rail, UART‑linked to the S3. |
| 🐧 **ARM Linux co‑processor** | Luckfox RV1106 (Core1106) | A genuine Linux computer on the shield for capture, monitoring and heavy tooling. Self‑powered from **its own USB‑C**. |
| 🧲 **NFC** (13.56 MHz) | PN532 | Read / emulate / relay over a **dedicated I²C bus** owned by the S3 — independent of the radios. |
| 📇 **125 kHz LF RFID** | HTRC110 + coil | Low‑frequency prox‑card read &amp; emulation (EM4100‑class) — a band the stock board lacks. |
| 🔑 **iButton / 1‑Wire** | 1‑Wire header | Dallas iButton + 1‑Wire contact interface. |
| 📻 **LoRa / long‑range Sub‑GHz** | SX1262 (+ Johanson match) | Long‑range LoRa &amp; FSK on Sub‑GHz, alongside the OOK/ASK radios. |
| 📻 **High‑performance Sub‑GHz** | CC1200 | A higher‑performance Sub‑GHz transceiver in addition to the classic CC1101. |
| 💾 **External flash** | W25Q128 (16 MB) | Non‑volatile storage for captures, logs and dumps. |
| 🧱 **4‑layer, power‑hardened PCB** | — | Dedicated **GND** and **3V3** planes, per‑rail decoupling and bulk caps for clean RF and stable multi‑radio operation. |

---

## 🎯 Features

SnowOwl runs the full **ESP32‑DIV** firmware feature set on the ESP32‑S3, extended with the SnowOwl radios and readers. Capabilities by band:

### 📡 Wi‑Fi — 2.4 GHz (ESP32‑S3) **+ 5 GHz (ESP32‑C5)** · **5 GHz is NEW**
| Capability | Notes |
|---|---|
| Dual‑band recon &amp; scanning | AP / station discovery across **2.4 *and* 5 GHz**, channels, RSSI |
| Deauth / attack testing | Targeted &amp; broadcast frame testing, now on 5 GHz networks too |
| Beacon / probe flooding | SSID beacon spam, probe generation |
| Packet monitor / capture | Sniffing, with hand‑off to the ARM side for storage &amp; analysis |
| Evil‑portal / captive testing | Rogue‑AP and captive‑portal workflows |

### 🔵 Bluetooth / BLE
| Capability | Notes |
|---|---|
| BLE scan &amp; recon | Device discovery, service enumeration |
| Advertising / spam | BLE advertisement flooding &amp; spoofing |
| Proximity‑pairing tests | Device‑spam nuisance testing |

### 📶 2.4 GHz / nRF24 (3× nRF24L01+)
| Capability | Notes |
|---|---|
| Channel scanner / analyzer | 2.4 GHz spectrum sweep across three radios |
| Multi‑channel jamming (test) | Wide/targeted 2.4 GHz interference testing |

### 📻 Sub‑GHz — CC1101 · CC1200 · **SX1262 LoRa**
| Capability | Notes |
|---|---|
| Scan &amp; frequency analysis | RSSI sweep, signal hunting |
| Capture / replay | Record &amp; re‑transmit OOK/ASK signals |
| **LoRa / long‑range** | SX1262 for LoRa &amp; long‑range FSK |
| Jamming (test) | Sub‑GHz interference testing |

### 📺 Infrared (TSOP1838 RX + IR TX)
| Capability | Notes |
|---|---|
| Receive / decode | Capture IR remotes |
| Send / replay | Transmit stored or brute‑forced codes |

### 🧲 NFC / RFID — 13.56 MHz (PN532)
| Capability | Notes |
|---|---|
| Read / dump | MIFARE‑class tags &amp; UIDs |
| Emulate / relay | Card emulation and relay over the dedicated I²C bus |

### 📇 LF RFID — 125 kHz (HTRC110) &amp; 🔑 iButton · **NEW**
| Capability | Notes |
|---|---|
| 125 kHz read / emulate | EM4100‑class prox cards |
| iButton / 1‑Wire | Dallas contact keys |

### 🛰️ GPS (NEO‑M9N)
| Capability | Notes |
|---|---|
| Location &amp; time | Fix, NMEA, wardriving support |

### 🐧 ARM Linux co‑processor (RV1106) · **NEW**
| Capability | Notes |
|---|---|
| On‑device capture &amp; storage | Long‑running packet/loot capture without a laptop, to the 16 MB flash / SD |
| Monitoring / blue‑team | Run standard Linux security &amp; monitoring tooling on‑device |
| Scripting &amp; automation | Python/shell orchestration of the S3 &amp; C5 radios |

### 🧰 Device &amp; System
| Capability | Notes |
|---|---|
| Display UI | On‑device menu on the ESP32‑DIV TFT |
| Storage | microSD + **16 MB external flash** |
| Settings | Per‑module configuration |

> The exact per‑tool menu is inherited from the ESP32‑DIV firmware — see the [ESP32‑DIV wiki](https://github.com/cifertech/ESP32-DIV/wiki) for the full tool list.

---

## 🔧 Hardware Overview

### 🧠 Compute
- **ESP32‑S3** — application processor, 2.4 GHz Wi‑Fi &amp; BLE, UI (on the main board)
- **ESP32‑C5‑WROOM‑1U** — dual‑band **2.4 / 5 GHz Wi‑Fi 6** co‑processor (U.FL antenna, AP7361C LDO rail, UART link)
- **Luckfox RV1106 (Core1106)** — ARM Linux SoM, self‑powered via its **own USB‑C**

### 📡 Radios
- **3× nRF24L01+** — 2.4 GHz
- **CC1101** + **CC1200** — Sub‑GHz transceivers
- **SX1262** (+ Johanson integrated match) — LoRa / long‑range Sub‑GHz
- **7× SMA** antenna ports + U.FL

### 🧲 Readers &amp; sensors
- **PN532** — 13.56 MHz NFC (dedicated I²C bus, 27.12 MHz)
- **HTRC110** + coil — 125 kHz LF RFID
- **iButton / 1‑Wire** contact interface
- **TSOP1838** IR receiver + IR transmit
- **NEO‑M9N** — GPS

### 🔌 Power, storage &amp; protection
- **W25Q128** — 16 MB SPI flash
- microSD
- **AP7361C‑3.3** LDO (C5 rail), dedicated USB‑C for the ARM SoM
- **USBLC6‑2SC6** USB ESD protection, **SN74LVC2G34** level buffer
- Power hardening: bulk + per‑rail decoupling capacitors

Mates to the main board through the existing 20‑pin stack header; pogo/standoff holes are kept clear for a flat stack.

---

## 🧱 PCB / Manufacturing

- Designed in **KiCad 10**.
- **4‑layer stackup** — `F.Cu (signal) · In1 (GND plane) · In2 (3V3 plane) · B.Cu (signal)` for clean RF return paths and a quiet supply.
- Power‑planned per the datasheets of each part (per‑rail decoupling + bulk); RF nets use integrated matching where the transceiver vendors provide it.
- Intended for standard 4‑layer fabrication (e.g. JLCPCB); the 4‑layer option adds only a few dollars per unit over 2‑layer.

> **Status:** hardware design in active development. Placement and power are complete; signal routing is being finished. Renders and design notes live in [`/hardware`](hardware).

---

## ⚠️ Legal &amp; Responsible Use

SnowOwl is an **offensive *and* defensive** research platform. It is intended **only** for:

- testing systems you **own** or are **explicitly authorized** to assess,
- security education, CTFs, and lab research,
- defensive monitoring and blue‑team work.

Do **not** use it to access, disrupt, or interfere with networks, devices, cards, or systems you do not own or lack permission to test. Radio transmission is regulated — respect your local RF laws and licensed bands. **You** are responsible for how you use this tool.

---

## 📜 License

Released under the **MIT License** — see [`LICENSE`](LICENSE).

SnowOwl is a derivative of **ESP32‑DIV** (© CiferTech, MIT). The original work's terms are honored and its authors are credited below.

---

## 🙌 Credits

- **[CiferTech](https://github.com/cifertech)** — the original [ESP32‑DIV](https://github.com/cifertech/ESP32-DIV) platform and firmware that SnowOwl builds on. 🙏
- **YurilLAB** — SnowOwl hardware: the 5 GHz Wi‑Fi + ARM‑Linux compute, the NFC / 125 kHz RFID / iButton / LoRa additions, power hardening, and the 4‑layer redesign.

---

## 💬 Contributing &amp; Support

Issues, ideas, and pull requests are welcome. If SnowOwl is useful to you, a ⭐ helps.
