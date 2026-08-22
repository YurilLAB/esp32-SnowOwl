<div align="center">

# 🦉 ESP32-SnowOwl

**A multi-purpose wireless _offensive &amp; defensive_ security multitool.**

ESP32-S3 **+** an on-board ARM Linux co-processor — with NFC, 125 kHz RFID, iButton, Sub-GHz, BLE, Wi‑Fi, IR &amp; GPS, in one stackable handheld.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Platform](https://img.shields.io/badge/platform-ESP32--S3%20%2B%20ARM%20Linux-blue)
![PCB](https://img.shields.io/badge/PCB-KiCad%2010%20%C2%B7%204--layer-orange)
![Status](https://img.shields.io/badge/status-in%20development-yellow)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)

<img src="images/shield-front.png" alt="ESP32-SnowOwl shield" width="66%">

</div>

> ⚠️ **Authorized use only.** SnowOwl is built for security research, education, CTFs, and testing on systems you own or are explicitly authorized to assess. You are solely responsible for complying with all applicable laws and regulations.

---

## 📖 About

**ESP32-SnowOwl** is a hardware evolution of [CiferTech's **ESP32‑DIV**](https://github.com/cifertech/ESP32-DIV) — the open-source ESP32‑S3 wireless multitool. SnowOwl keeps the entire ESP32‑DIV toolset and gives it a **second brain**: a full **ARM Linux co‑processor** for heavyweight, on‑device tooling, plus **NFC**, **125 kHz LF RFID**, and **iButton / 1‑Wire**.

That two‑brain split is the whole idea:

- 🦅 **ESP32‑S3 — the talons.** Real‑time wireless **offense**: Wi‑Fi, BLE, 2.4 GHz / nRF24, Sub‑GHz, IR, and 13.56 MHz NFC.
- 🐧 **ARM Linux (RV1106) — the eyes.** A real Linux userland for **defense &amp; analysis**: packet capture, monitoring, scripting, and standard Linux security tooling running right on the device.

Silent, patient, and built for the dark — the snow owl fits.

---

## 🦉 What SnowOwl adds over ESP32‑DIV

| Addition | What it brings |
|---|---|
| 🐧 **ARM Linux co‑processor** — Luckfox RV1106 SoM | A genuine Linux computer on the shield for capture, monitoring and heavy tooling the ESP32 can't run. Self‑powered from **its own USB‑C**. |
| 🧲 **PN532 NFC** (13.56 MHz) | Read / emulate / relay NFC over a **dedicated I²C bus** owned by the S3 — independent of the radios. |
| 📇 **125 kHz LF RFID** | Low‑frequency prox‑card read &amp; emulation (EM4100‑class) — the band the stock board lacks. |
| 🔑 **iButton / 1‑Wire** | Dallas iButton + 1‑Wire contact interface. |
| 💾 **External flash** | Extra non‑volatile storage for captures, logs and dumps. |
| 🧱 **4‑layer, power‑hardened PCB** | Dedicated **GND** and **3V3** planes, per‑rail decoupling and bulk capacitors for clean RF and stable multi‑radio operation. |

---

## 🎯 Features

SnowOwl runs the full **ESP32‑DIV** firmware feature set on the ESP32‑S3, extended with the SnowOwl peripherals. Capabilities by band:

### 📡 Wi‑Fi
| Capability | Notes |
|---|---|
| Recon &amp; scanning | AP / station discovery, channels, RSSI, vendor lookup |
| Deauth / attack testing | Targeted and broadcast frame testing |
| Beacon / probe flooding | SSID beacon spam, probe generation |
| Packet monitor / capture | Sniffing, with hand‑off to the ARM side for storage &amp; analysis |
| Evil‑portal / captive testing | Rogue‑AP and captive‑portal workflows |

### 🔵 Bluetooth / BLE
| Capability | Notes |
|---|---|
| BLE scan &amp; recon | Device discovery, service enumeration |
| Advertising / spam | BLE advertisement flooding &amp; spoofing |
| Sour‑Apple / device‑spam tests | Proximity‑pairing nuisance testing |

### 📶 2.4 GHz / nRF24
| Capability | Notes |
|---|---|
| Channel scanner / analyzer | 2.4 GHz spectrum sweep |
| Jamming (test) | Wide/targeted 2.4 GHz interference testing |

### 📻 Sub‑GHz (CC1101 / CC1200)
| Capability | Notes |
|---|---|
| Scan &amp; frequency analysis | RSSI sweep, signal hunting |
| Capture / replay | Record and re‑transmit OOK/ASK signals |
| Jamming (test) | Sub‑GHz interference testing |

### 📺 Infrared
| Capability | Notes |
|---|---|
| Receive / decode | Capture IR remotes |
| Send / replay | Transmit stored or brute‑forced codes |

### 🧲 NFC / RFID — 13.56 MHz (PN532)
| Capability | Notes |
|---|---|
| Read / dump | MIFARE‑class tags &amp; UIDs |
| Emulate / relay | Card emulation and relay over the dedicated I²C bus |

### 📇 LF RFID — 125 kHz &amp; 🔑 iButton  ·  **NEW**
| Capability | Notes |
|---|---|
| 125 kHz read / emulate | EM4100‑class prox cards |
| iButton / 1‑Wire | Dallas contact keys |

### 🛰️ GPS
| Capability | Notes |
|---|---|
| Location &amp; time | Fix, NMEA, wardriving support |

### 🐧 ARM Linux co‑processor  ·  **NEW**
| Capability | Notes |
|---|---|
| On‑device capture &amp; storage | Long‑running packet/loot capture without a laptop |
| Monitoring / blue‑team | Run standard Linux security &amp; monitoring tooling on‑device |
| Scripting &amp; automation | Python/shell orchestration of the S3's radios |

### 🧰 Device &amp; System
| Capability | Notes |
|---|---|
| Display UI | On‑device menu on the ESP32‑DIV TFT |
| Storage | microSD + external flash |
| Settings | Per‑module configuration |

> The exact per‑tool menu is inherited from the ESP32‑DIV firmware — see the [ESP32‑DIV wiki](https://github.com/cifertech/ESP32-DIV/wiki) for the full tool list.

---

## 🔧 Hardware Overview

### 🧠 Main Board
- **ESP32‑S3** application processor + 2.4 GHz Wi‑Fi / BLE
- **ILI9341** SPI TFT display + navigation buttons
- **CC1101 / CC1200** Sub‑GHz transceiver(s) with SMA
- **nRF24L01+** 2.4 GHz module
- **IR** transmit / receive
- **GPS** module (NEO‑class)
- microSD, USB‑C, LiPo charging + power management

### 🛡️ SnowOwl Shield
- **Luckfox RV1106** ARM Linux SoM — self‑powered via its **own USB‑C**
- **PN532** NFC (dedicated I²C bus)
- **125 kHz LF RFID** front‑end + coil connector
- **iButton / 1‑Wire** contact interface
- **External flash** + power‑hardening (bulk + decoupling caps)
- Mates to the main board through the existing stack header + pogo/standoff holes

---

## 🧱 PCB / Manufacturing

- Designed in **KiCad 10**.
- **4‑layer stackup** — `F.Cu (signal) · In1 (GND plane) · In2 (3V3 plane) · B.Cu (signal)` for clean RF return paths and a quiet supply.
- Power‑planned per the datasheets of each part (per‑rail decoupling + bulk).
- Intended for standard 4‑layer fabrication (e.g. JLCPCB); the 4‑layer option adds only a few dollars per unit over 2‑layer.

> **Status:** hardware design in active development. Placement and power are complete; final signal routing is in progress. Renders and design notes live in [`/hardware`](hardware).

---

## ⚠️ Legal &amp; Responsible Use

SnowOwl is an **offensive *and* defensive** research platform. It is intended **only** for:

- testing systems you **own** or are **explicitly authorized** to assess,
- security education, CTFs, and lab research,
- defensive monitoring and blue‑team work.

Do **not** use it to access, disrupt, or interfere with networks, devices, cards, or systems you do not own or lack permission to test. Radio transmission is regulated — respect your local RF laws. **You** are responsible for how you use this tool.

---

## 📜 License

Released under the **MIT License** — see [`LICENSE`](LICENSE).

SnowOwl is a derivative of **ESP32‑DIV** (© CiferTech, MIT). The original work's terms are honored and its authors are credited below.

---

## 🙌 Credits

- **[CiferTech](https://github.com/cifertech)** — the original [ESP32‑DIV](https://github.com/cifertech/ESP32-DIV) platform and firmware that SnowOwl builds on. 🙏
- **YurilLAB** — SnowOwl hardware: the ARM‑Linux + NFC/LF‑RFID/iButton shield, power hardening, and the 4‑layer redesign.

---

## 💬 Contributing &amp; Support

Issues, ideas, and pull requests are welcome. If SnowOwl is useful to you, a ⭐ helps.
