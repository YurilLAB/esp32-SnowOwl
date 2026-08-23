# SnowOwl v2 — Placement & Floorplan Strategy

> The whole reason for the v2 remake: **decide where every block lives *before* we
> route**, so signals travel a short straight line from source to load and nothing
> has to snake across the whole board. This document is the contract the schematic
> partition and the board zones both follow.

**Form factor:** stackable **2-board** set. `main-board` (ESP32-S3 brain + UI) on the
bottom, `shield` (co-processors + radios + readers) on top, mated by a single
**2x10 2.54 mm stack header (J1)**. Both boards share the same outline, the same
mounting pattern, and the **same J1 position** so they align.

**Outline:** `70 x 100 mm`, 4x M3 mounting holes 4 mm in from the corners, J1 datum at
`(53.57, 106.0)` on both boards. This was grown from a first 60x85 target after a
footprint fit-check: the RV1106 SoM (about 32 x 33 mm), the ESP32-C5 module and the
three nRF24 modules did not fit 60x85 without cramming, which is exactly what forces
congested traces. 70x100 is still far smaller than v1 (99 x 135) and leaves real
routing channels between the zones. If you have an enclosure limit, give me the max
X/Y and I will reflow to fit.

---

## The three placement rules

1. **Antennas and connectors own the edges.** Every RF part sits immediately inboard of
   its own antenna, SMA or coil on the board edge, so the RF trace is under about 10 mm
   and never crosses another radio. User I/O (USB-C, microSD, iButton, IR) also lives on
   the edge nearest its zone.
2. **The bus is a spine, and it is mastered on the shield.** The **ESP32-C5**, which sits
   among the radios, masters one short central **SPI spine**; every SPI radio taps off it
   with a short stub instead of a run back to a far MCU. The main-board ESP32-S3 does
   **not** reach the radios directly. It commands the C5 over a UART link across J1. This
   keeps the entire radio fan-out on the shield, next to the C5, so nothing long crosses
   the stack header.
3. **Big modules go on the back.** The tall or large parts (3x nRF24, the RV1106 SoM) sit
   on the bottom side so the outline stays small and the front stays routable.

---

## Who owns what (the architecture that keeps traces short)

| Controller | Where | Owns | Talks to the S3 by |
|---|---|---|---|
| **ESP32-S3** | main board | UI: TFT, LEDs, nav buttons, microSD, and (over J1) the NFC reader, iButton and IR | it is the brain |
| **ESP32-C5** | shield, centre | the SPI radio spine: CC1101, CC1200, SX1262, 3x nRF24, W25Q128 flash; plus 5 GHz Wi-Fi and the 125 kHz HTRC110 | one UART pair + a data-ready line on J1 |
| **RV1106** | shield, back | heavy Linux tooling; owns its own USB-C, microSD and (optionally) the GPS | one UART pair + a control line on J1 |

Consequence for routing: all the SPI chip-selects, CE and IRQ lines stay on the shield
between the C5 and the radios. Only power and two thin comm links cross J1.

**C5 GPIO budget.** Driving the sub-GHz trio, three nRF24 and the flash off one C5 is a
lot of chip-select, CE and IRQ lines. If the C5 runs short of pins we add a small
CS/IO expander (a 74HC595 for chip-selects, or an SPI/I2C expander) placed **on the
spine, on the shield**, so even that stays local. The `03_ESP32-C5` zone leaves room
for it. This is a shield-only detail now, it never touches J1.

---

## Main board (brain) — zone map — phone style

Front is pure UI; all electronics sit on the back, with the display turned landscape.

```
   FRONT (UI side)                 BACK (electronics)
   +------------------+            +------------------+
   |   STATUS LEDs    |            |                  |
   | +--------------+ |            |   [ ESP32-S3 ]   |  S3 behind the display
   | |  TFT  land-  | |            |     = HUB    uSD |  microSD by the S3 (east)
   | |  scape       | |            |                  |
   | +--------------+ |            |                  |
   |   NAV BUTTONS    |            | POWER  ..J1..     |  J1 = horizontal strip
   |   ....J1....     |            |     [USB-C]       |  USB-C bottom-centre
   +------------------+            +------------------+
```

| Sheet / zone | Side, where | Why here |
|---|---|---|
| `03_Display_UI` | front | LEDs top, landscape TFT centre, nav buttons below. The whole front is UI. |
| `02_ESP32-S3` | back, upper-centre | Directly behind the display; display bus vias straight through, other buses drop to J1 below. Antenna keepout to the nearest corner. |
| `04_microSD` | back, east | Beside the S3, slot out the right edge. |
| `01_Power_USB` | back, bottom-centre | USB-C at the bottom edge like a phone, LDO and ESD next to it, away from the S3 and display. |
| `05_Stack_Header` (J1) | lower-centre, horizontal | Between the nav buttons and the USB-C so nothing fights for the centre column. Same X/Y on both boards. |

## Shield (expansion) — zone map

```
            N (top edge = SMA row)
   [SMA1] [SMA2] [SMA3] [SMA4]
   +----------------------------------+
   |CC1101| CC1200 | SX1262 |  GPS     |  each Sub-GHz + GPS behind its own SMA
   |------+--------+--------+----------|  (nRF24 x3 sit on the BACK, right behind)
   |PN532 | flash  |                   |
   |+coil |  IR    |   ESP32-C5        |E   C5 = SPI master, U.FL out the EAST edge
 W |(I2C) |        |   (SPI MASTER)    |
   |iButton|      ..J1 STACK..         |    RV1106 + HTRC110 on the BACK
   +----------------------------------+
            S (bottom edge)
```

| Sheet / zone | Side | Where | Why here |
|---|---|---|---|
| `05_SubGHz` (CC1101 / CC1200 / SX1262) | F | north band, 3 slots | Each transceiver hugs its SMA on the top edge; all three tap the C5 SPI spine just below. |
| `10_GPS` (NEO-M9N) | F | NE corner | Clear-sky antenna at the corner; UART to the C5 or RV1106, away from the TX radios. |
| `03_ESP32-C5` | F | centre-east, SPI master | Sits under the radio band so the spine is short in every direction; U.FL out the east edge; leaves room for a CS expander. |
| `07_NFC_PN532` | F | west | 13.56 MHz coil out the west edge on its own I2C, isolated from the radios. |
| `12_Flash` (W25Q128) | F | centre, on the spine | Tiny SPI part next to the C5. |
| `11_IR` | F | centre | TSOP1838 + IR TX, line of sight to the edge. |
| `09_iButton_1Wire` | F | west edge | Contact probe at the edge. |
| `06_nRF24 x3` | B | north, behind the Sub-GHz | Grouped over the spine, shared SCK/MOSI/MISO, per-radio CE/CSN/IRQ, short via drop to the C5. |
| `04_RV1106` (Luckfox SoM) | B | centre-east | Biggest part; own USB-C and microSD out the east edge; UART to J1 below. |
| `08_LF_RFID_HTRC110` | B | west-mid | 125 kHz coil out the west edge, offset in Y from the NFC coil to cut coupling (the two readers never run at once). |
| `02_Power` | B | south, below J1 | Bulk caps under the LDOs. RV1106 self-powered from its own USB-C. |

---

## J1 stack-header pinout (20 pins, the anti cross-trace move)

With the C5 owning the radios, J1 no longer carries any SPI radio lines. It carries
power plus the two comm links and the few UI peripherals the S3 keeps. Pin order is
chosen so each signal lands next to the zone that uses it on both boards.

| Pin | Net | Pin | Net |
|---|---|---|---|
| 1 | `+5V` | 11 | `GND` |
| 2 | `+3V3` | 12 | `OW_1WIRE` (iButton) |
| 3 | `GND` | 13 | `IR_RX` |
| 4 | `SDA` (PN532) | 14 | `IR_TX` |
| 5 | `SCL` (PN532) | 15 | `GND` |
| 6 | `GND` | 16 | `RV_TX` (to S3) |
| 7 | `C5_TX` (to S3) | 17 | `RV_RX` (from S3) |
| 8 | `C5_RX` (from S3) | 18 | `RV_CTRL` (boot/enable) |
| 9 | `C5_DRDY` (C5 to S3 IRQ) | 19 | `SPARE` |
| 10 | `GND` | 20 | `GND` |

Power and I2C are on the west pins (1 to 6), next to the shield power-in and the PN532.
The C5 link is centre-east (7 to 9), next to the C5. The RV1106 link is on the east
pins (16 to 18), next to the RV1106. Six grounds are spread through the connector for
clean returns and to separate the signal groups.

> This is the target pinout, not final silicon pin assignments. The exact ESP32-S3 and
> ESP32-C5 GPIO to J1 mapping is set when we wire `02_ESP32-S3` and `03_ESP32-C5`. The
> point is that the groups are contiguous and match the physical zones, so the fan-out
> on each board is short and un-crossed.

---

## Stackup (4-layer)

| Layer | Role |
|---|---|
| `F.Cu` | signal (short local routing within each zone) |
| `In1.Cu` | GND plane, unbroken reference under every RF part |
| `In2.Cu` | 3V3 plane |
| `B.Cu` | signal (back-side modules + spine returns) |

Dedicated GND and 3V3 planes are the single biggest win for a multi-radio board. The
plane names are set in each `.kicad_pcb` (`In1.Cu` to GND, `In2.Cu` to 3V3). When the
copper goes in, type these as plane pours and stitch them.

---

## What is / is not done

**Done:**
- Both KiCad projects open at 70 x 100 mm, 4-layer, with mounting holes and the shared
  J1 datum placed identically on both boards.
- Floorplan zones drawn on the board (`User.1` front, `User.2` back), sized to the real
  part footprints, positioned for the C5-owns-radios plan, with a matching SVG per board.
- Hierarchical schematics partitioned one sheet per subsystem, each naming its parts,
  nets and board zone.
- Architecture and J1 pinout locked (this document).

**Next:**
1. Assemble the symbol library set (harvest the parts already drawn on v1, pull in the
   Espressif and vendor symbols, draw the few that do not exist).
2. Drop real symbols into each schematic sheet and wire the buses to the J1 pinout above.
3. Set the final ESP32-S3 and ESP32-C5 GPIO to J1 mapping and the C5 CS/IRQ plan.
4. Place real footprints inside each zone, add the ground-plane pours and stitching.
5. Hand-route the RF nets (matched, short), then the C5 spine, then everything else.
