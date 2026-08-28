# SnowOwl / ESP32-BlackRaven — component & schematic research

Electrical reference carried over from the earlier attempt. **All CAD (boards, schematics,
floorplan, renders) was wiped and is being redone** — the project direction is changing. What
survives here is only the research worth not repeating: what the parts are, how they connect,
and the datasheet findings.

Nothing here assumes a particular board size, layout or form factor.

---

## 1. System architecture

| Controller | Owns | Link |
|---|---|---|
| **ESP32-S3** | UI: TFT, LEDs, nav buttons, microSD; and NFC, iButton, IR | the brain |
| **ESP32-C5** | the SPI radio spine — CC1101, CC1200, SX1262, 3× nRF24, W25Q128 flash; plus 5GHz Wi-Fi and the 125kHz HTRC110 | UART pair + data-ready line to the S3 |
| **RV1106** | ARM Linux SoM, own microSD | UART to the S3 |

**Key decision:** the C5 sits among the radios and masters one short SPI spine; every radio taps
it with a short stub. The S3 never reaches the radios directly — it commands the C5 over UART.
An **MCP23S17** expander generates the per-radio chip-selects so the C5 doesn't burn GPIO.

## 2. Component inventory

| Function | Part | Notes |
|---|---|---|
| Main MCU | ESP32-S3 | UI + brain |
| Radio co-processor | ESP32-C5-WROOM-1U | 5GHz Wi-Fi; **module — own antenna** |
| CS expander | MCP23S17 | SPI → chip-selects |
| Sub-GHz | **CC1101** | **plug-in module** (2×4 header) |
| Sub-GHz | **CC1200** | chip-down, needs 40MHz xtal + RF front-end |
| LoRa | **SX1262** | chip-down, needs 32MHz xtal + RF front-end |
| 2.4GHz | 3× nRF24L01+ | **modules — own antennas** |
| NFC | PN532 | 13.56MHz |
| LF RFID | HTRC110 | 125kHz, **5V part**, open-drain DOUT pulled to 3V3 — level-shift care needed vs the 3.3V C5 |
| GPS | u-blox NEO-M9N | chip-down, needs antenna bias-tee |
| IR | receiver + TX LED + transistor | |
| iButton | 1-Wire | |
| Flash | W25Q128 | 16MB, on the SPI spine |
| Linux | RV1106 "Core1106" SoM | + own microSD |
| Display | TFT on FPC header | |

### Radios that need on-board RF vs. those that don't
- **Need a front-end (chip-down):** CC1200, SX1262, GPS NEO-M9N.
- **Self-contained (own crystal + antenna):** nRF24 ×3, CC1101, ESP32-C5-WROOM-1U.

> **CC1101 is a module, not a chip** — a 2×4 pin header: GND, +3V3, SPI SCK/MOSI/MISO, CSN,
> GDO0, GDO2. It needs no crystal and no RF path, only decoupling. (This confused us once.)

## 3. Power

**One USB-C for the whole device.** Everything runs off the battery; USB only charges.

```
USB-C → VBUS → BQ25896 charger ─┬→ VBAT  (LiPo ~2500mAh, ≈1–2h active runtime)
                                 └→ VSYS ─┬→ TPS63020 buck-boost → +3V3
                                          └→ TPS61230A boost     → +5V
```
Plus **MAX17048** fuel gauge and **DW01A + FS8205A** cell protection, both readable over I²C.
The RV1106 has microSD but no USB port of its own — reached over UART.

### ⚠ Datasheet pinout corrections (researched, never applied — the old symbols were WRONG)

- **BQ25896** (QFN-24): has **no D+/D− pins** — it uses **PSEL (2)** and **PG (3)**.
  VBUS=1, STAT=4, SCL=5, SDA=6, INT=7, OTG=8, CE=9, ILIM=10, TS=11, QON=12, BAT=13/14,
  SYS=15/16, PGND=17/18+EP, SW=19/20, BTST=21, REGN=22, PMID=23, NC=24.
  App values: inductor **1µH**, R_ILIM ≈ **180Ω** for 2A, TS divider **10k/10k**,
  charge current set over I²C (2048mA).
- **TPS63020**: **14-pin VSON**, not 10-pin. Inductor **1.5µH**, FB **0.5V**,
  3.3V divider **R1=1M / R2=180k**.
- **TPS61230A**: **7-pin 2×2 VQFN** — CBST=1, EN=2, FB=3, VIN=4, SW=5, GND=6, VOUT=7.
  Inductor **1µH**, FB **1.2V**, 5V divider **R1=316k / R2=100k**.
- **MAX17048**: separate **CELL (2)** and **VDD (3)** pins.
- USB-C **5.1k CC resistors** — confirmed correct.

## 4. RF front-end designs

**All L/C values below are FIRST-PASS starting points — they must be tuned with a VNA** against
each chip's reference design and the final board's 50Ω trace impedance.

| Radio | Chain |
|---|---|
| **CC1200** | PA + LNA lumped match → 2-section harmonic LPF → antenna |
| **SX1262** | DC-DC coil (≈15µH) + PA-regulator cap; RFO/RFI match → 2-section LPF → antenna |
| **GPS** | active-antenna bias-tee: series DC-block cap + DC feed inductor/ferrite from the RF supply |

Edge-mount SMA convention: **pin 1 = signal, pin 2 = GND shield**, barrel pointing off the edge.

⚠ **The CC1200 RF pinout (PA / TRXSW / LNA_P / LNA_N / LPF0 / LPF1) was never datasheet-verified.**
Confirm it before designing that match again.

## 5. Schematic gotchas worth remembering

- **eeschema has no Python API.** Schematics have to be hand-authored as S-expressions (or drawn
  by hand); `kicad-cli sch export netlist` is what drives everything downstream.
- **Auto-generated stub collisions silently merge nets.** When a generator drops a short wire stub
  + net label at each pin, packing parts into a 2-D grid can land a shunt-to-GND stub exactly on a
  signal stub — fusing whole nets with no error. It bit us twice: once shorting GND to a decoupling
  net, once merging *every* antenna net into GND. **Lay each network out as a single well-spaced
  row with distinct X positions**, and always re-verify net node counts after generating — a rail
  with a suspiciously large node count means a merge happened.
- **Verify a symbol's pin count against the datasheet before trusting it** — several power ICs
  above were placeholder symbols with invented pinouts, and it wasn't caught until much later.
