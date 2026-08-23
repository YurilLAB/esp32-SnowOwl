# 🦉 ESP32-SnowOwl — Hardware

KiCad design for the SnowOwl 2-board stack (main board + shield).

## PCB source → [`../PCB/`](../PCB)

All CAD lives under [`../PCB/`](../PCB), with old and new kept separate:

- **[`PCB/v2-compact/`](../PCB/v2-compact)** — the **active** ground-up remake: a compact,
  deliberately floorplanned **2-board stack** (`main-board/` + `shield/`). Start with
  [`PCB/v2-compact/PLACEMENT.md`](../PCB/v2-compact/PLACEMENT.md) for the zone map and the
  bus/routing strategy.
- **[`PCB/v1-original/`](../PCB/v1-original)** — the original board, archived (renders only).

## Renders (v1)

| Front | Back |
|---|---|
| ![front](../images/shield-front.png) | ![back](../images/shield-back.png) |

## Shield highlights

- **ESP32-C5-WROOM-1U** — dual-band **2.4 / 5 GHz Wi-Fi 6** co-processor (adds the 5 GHz band the S3 lacks); own U.FL antenna + **AP7361C-3.3** LDO rail, UART-linked to the S3.
- **Luckfox RV1106 (Core1106)** ARM Linux SoM — self-powered from its **own USB-C** (isolated from the main board's supply).
- **PN532 NFC** (13.56 MHz, 27.12 MHz clock) on a dedicated I²C bus owned by the ESP32-S3.
- **HTRC110 125 kHz LF RFID** front-end + coil connector.
- **iButton / 1-Wire** contact interface.
- **SX1262 LoRa** (+ Johanson integrated match) and **CC1200** high-performance Sub-GHz, alongside the classic CC1101.
- **3× nRF24L01+** (2.4 GHz) and **NEO-M9N** GPS.
- **W25Q128** 16 MB external flash for captures/logs.
- Power hardening: bulk + per-rail decoupling capacitors; **USBLC6** USB ESD protection.
- Mates to the main board via the existing 20-pin stack header; pogo/standoff holes kept clear for a flat stack.

## Stackup (4-layer)

| Layer | Role |
|---|---|
| `F.Cu`  | Signal |
| `In1.Cu` | **GND plane** |
| `In2.Cu` | **3V3 plane** |
| `B.Cu`  | Signal |

Dedicated planes give every radio a solid return path and a quiet supply — the single biggest win for multi-radio RF behaviour. Standard 4-layer fabrication (e.g. JLCPCB) adds only a few dollars per unit over 2-layer.

## Status

**Redesign in progress — v2 (2-board stack).** The board is being remade from scratch to
fix v1's spread-out placement (which forced long cross-board traces). v2 keeps every
radio/reader but floorplans each subsystem deliberately so routing stays local.

Current v2 state (in [`../PCB/v2-compact/`](../PCB/v2-compact)): floorplan **skeleton** —
both KiCad projects open with the compact `60 × 85 mm` outline, the 4-layer stackup, the
shared 20-pin `J1` stack datum, mounting holes, and labelled placement zones. Symbols,
footprints and routing are the next passes. The RF nets (Sub-GHz, NFC coil, GPS) are still
reserved for short, matched hand-routing.
