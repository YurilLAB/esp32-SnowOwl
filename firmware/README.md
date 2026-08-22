# 🦉 ESP32-SnowOwl — Firmware

SnowOwl is a **three-processor** device, so there are three firmware sides.

## 🦅 ESP32-S3 (controller + main radios)

- **Base:** the [ESP32-DIV firmware](https://github.com/cifertech/ESP32-DIV) (Arduino / PlatformIO) — the full wireless toolset runs unchanged.
- **SnowOwl add-ons:**
  - **PN532 NFC** over the dedicated I²C bus.
  - **HTRC110 125 kHz LF RFID** read / emulate.
  - **iButton / 1-Wire** contact interface.
  - **SX1262 LoRa** + **CC1200** Sub-GHz drivers.

Flashing follows the ESP32-DIV instructions (USB-C, `esptool` / PlatformIO, or the browser flasher).

## 📡 ESP32-C5 (5 GHz Wi-Fi co-processor)

- Runs its own ESP-IDF firmware and is **UART-linked to the S3**, which requests 5 GHz scans/attacks and receives results to draw on the TFT.
- Adds the dual-band **2.4 / 5 GHz Wi-Fi 6** capability the S3 (2.4 GHz only) can't provide.
- Flashed over its own USB / `C5_PROG_UART` header.

## 🐧 RV1106 (ARM Linux co-processor)

- Runs a standard Linux userland (Luckfox Buildroot / Ubuntu image) from its own storage, powered by its own USB-C.
- Talks to the ESP32-S3 over the inter-processor link for capture, monitoring and orchestration.
- Intended home for heavier tooling the ESP32 can't run — packet capture, analysis, scripting, blue-team monitoring.

## Status

Firmware integration is in progress. Until the SnowOwl builds are published here, use the upstream [ESP32-DIV firmware + wiki](https://github.com/cifertech/ESP32-DIV/wiki) as the reference for the base toolset.
