# 🦉 ESP32-SnowOwl — Firmware

SnowOwl is a **two-brain** device, so there are two firmware sides.

## 🦅 ESP32-S3 (main radios)

- **Base:** the [ESP32-DIV firmware](https://github.com/cifertech/ESP32-DIV) (Arduino / PlatformIO) — the full wireless toolset runs unchanged.
- **SnowOwl add-ons:**
  - **PN532 NFC** over the dedicated I²C bus.
  - **125 kHz LF RFID** read / emulate.
  - **iButton / 1-Wire** contact interface.

Flashing follows the ESP32-DIV instructions (USB-C, `esptool` / PlatformIO, or the browser flasher).

## 🐧 RV1106 (ARM Linux co-processor)

- Runs a standard Linux userland (Luckfox Buildroot / Ubuntu image) from its own storage, powered by its own USB-C.
- Talks to the ESP32-S3 over the inter-processor link for capture, monitoring and orchestration.
- Intended home for heavier tooling the ESP32 can't run — packet capture, analysis, scripting, blue-team monitoring.

## Status

Firmware integration is in progress. Until the SnowOwl builds are published here, use the upstream [ESP32-DIV firmware + wiki](https://github.com/cifertech/ESP32-DIV/wiki) as the reference for the base toolset.
