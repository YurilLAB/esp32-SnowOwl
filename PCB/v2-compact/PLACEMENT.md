# SnowOwl v2 — Placement & Floorplan Strategy

> The whole reason for the v2 remake: **decide where every block lives *before* we
> route**, so signals travel a short straight line from source to load and nothing
> has to snake across the whole board. This document is the contract the schematic
> partition and the board zones both follow.

**Form factor:** stackable **2‑board** set — `main-board` (ESP32‑S3 brain + UI) on the
bottom, `shield` (co‑processors + radios + readers) on top — mated by a single
**2×10 2.54 mm stack header (J1)**. Both boards share the same outline, the same
mounting pattern, and the **same J1 position** so they align.

**Outline (starting target):** `60 × 85 mm`, 4× M3 mounting holes 4 mm in from the
corners. This is a deliberate shrink from the v1 render; it is a *starting* number —
easy to change (one rectangle + the hole positions in each `.kicad_pcb`, plus `W/H`
in the generator). If you have an enclosure constraint, give me the max X/Y and I'll
reflow the zones to fit.

---

## The three placement rules

1. **Antennas & connectors own the edges.** Every RF part sits immediately inboard of
   its own antenna/SMA/coil on the board edge, so the RF trace is < ~10 mm and never
   crosses another radio. User I/O (USB‑C, microSD, iButton, IR) also lives on the
   edge nearest its zone.
2. **The bus is a spine, not a star.** Shared SPI runs as one short central channel
   from J1; each SPI peripheral *taps off* the spine with a short stub instead of a
   dedicated run back to the MCU. I²C and UART branch off J1 toward their one/few
   consumers.
3. **Big modules go on the back.** The tall/large parts (3× nRF24, the RV1106 SoM)
   move to the bottom side so the outline stays small and the front stays routable.

---

## Main board (brain) — zone map — **phone-style**

The front is pure UI; all the electronics sit on the back — same rough layout as the
ESP32‑DIV but with the **display turned sideways (landscape)**.

```
   FRONT (UI side)                 BACK (electronics)
            N                              N
   +------------------+           +------------------+
   |   STATUS LEDs    |           |                  |
   | +--------------+ |           |  [ ESP32-S3 ]    |  S3 behind the display
   | |   TFT  (land-| |           |   = HUB          |
   | |   scape /    | |           |            uSD>  |  microSD by the S3 (east)
   | |   sideways)  | |           |                  |
   | +--------------+ |           |                  |
   |   NAV  BUTTONS   |           | POWER  ....J1.... |  J1 = horizontal strip
   |   ....J1....     |           |     [USB-C]       |  USB-C bottom-centre
   +------------------+           +------------------+
            S                              S
```

| Sheet / zone | Side · where | Why here |
|---|---|---|
| `03_Display_UI` | **front** | LEDs top, **landscape TFT** upper, **nav buttons below** (moved down). The whole front is UI, nothing else competes. |
| `02_ESP32-S3` | **back**, upper‑centre | On the back, directly behind the display; display bus vias straight through, other buses drop to J1 just below. Antenna keepout to the nearest corner edge. |
| `04_microSD` | **back**, east | Beside the S3 (both on the back); slot out the right edge. |
| `01_Power_USB` | **back**, bottom‑centre | **USB-C at the bottom-centre edge like a phone**; LDO/ESD next to it. Power stays at the bottom, away from the S3 and display. |
| `05_Stack_Header` (J1) | lower‑centre, **horizontal** | A 10×2 strip laid between the nav buttons and the USB-C so nothing fights for the centre column. Same X/Y on both boards. |

## Shield (expansion) — zone map

```
            N (top edge = SMA row)
   [SMA1][SMA2][SMA3][SMA4]
   +----------------------------------+
   |CC1101|CC1200|SX1262|  GPS         |  each Sub-GHz + GPS behind its own SMA
   |------+------+------+--------------|  (nRF24 x3 sit on the BACK, right behind)
   |PN532  | FLASH |   ESP32-C5        |
   |+ coil |  IR   |   + LDO (U.FL)  ->|E   C5 antenna out the EAST edge
W  |(I2C)  |       |                   |
   |iButton|   J1 STACK (SPI spine)    |    RV1106 + HTRC110 on the BACK
   +----------------------------------+
            S (bottom edge)
```

| Sheet / zone | Side | Where | Why here |
|---|---|---|---|
| `01_Stack_Header` (J1) | F | south, center | Same datum as the main board; buses arrive pre‑grouped (see pinout). |
| `05_SubGHz` (CC1101/CC1200/SX1262) | F | north band, 3 slots | Each transceiver hugs its SMA on the top edge; all three tap the SPI spine. |
| `10_GPS` (NEO‑M9N) | F | NE corner | Clear‑sky antenna at the corner; UART branch, away from the TX radios. |
| `03_ESP32-C5` | F | east | U.FL out the east edge, AP7361C rail local; only a UART pair back to J1. |
| `07_NFC_PN532` | F | west | 13.56 MHz coil out the west edge on its **own I²C** — isolated from the radios. |
| `12_Flash` (W25Q128) | F | center, on the spine | Tiny SPI part; sits on the SPI spine next to J1. |
| `11_IR` | F | center‑edge | TSOP1838 + IR TX need line‑of‑sight; two GPIO from J1. |
| `09_iButton_1Wire` | F | west edge | Contact probe at the edge; single GPIO. |
| `06_nRF24 ×3` | **B** | north, behind Sub‑GHz | Grouped over the spine, shared SCK/MOSI/MISO, per‑radio CE/CSN/IRQ. |
| `04_RV1106` (Luckfox SoM) | **B** | east‑mid, above J1 | Biggest part; own USB‑C + microSD at the edge; UART/IPC to J1 just below. |
| `08_LF_RFID_HTRC110` | **B** | west‑mid | 125 kHz coil out the edge, kept **diagonally opposite** the NFC coil to cut coupling. |
| `02_Power` | **B** | below J1 | Bulk caps under the LDOs. RV1106 self‑powered from its own USB‑C. |

---

## J1 stack-header pin grouping (the anti–cross-trace move)

The 20‑pin connector (a **horizontal 10×2 strip in the lower‑centre**) is the one thing
that crosses between boards, so its pin *order* is chosen so each signal lands next to
the zone that uses it — on **both** boards:

| Pins | Group | Lands near (shield) | Lands near (main) |
|---|---|---|---|
| 1, 2 | `+3V3`, `+5V` | power in | power out (west) |
| 3–4 | `GND` | — | — |
| 5–11 | **SPI block** — SCK, MOSI, MISO + a few CS/CE | central SPI spine | S3 south pins |
| 12–13 | `GND` (shield/return) | — | — |
| 14–15 | **I²C block** — SDA, SCL | PN532 (west) | S3 |
| 16–19 | **UART block** — C5 TX/RX, RV1106/GPS TX/RX | C5/RV1106/GPS (east) | S3 |
| 20 | `GND` | — | — |

> This is a *plan*, not final silicon pin assignments — the exact ESP32‑S3 GPIO ↔ J1
> mapping is set when we wire `02_ESP32-S3`. The point is that the **groups** are
> contiguous and ordered to match the physical zones, so the fan‑out on each board is
> short and un‑crossed.

### SPI chip-select / GPIO budget (why some radios may share or expand)

Keeping every radio means a lot of CS/IRQ lines. Counted out: nRF24 ×3 (3×CSN + 3×CE +
3×IRQ), CC1101 (CS + GDO0), CC1200 (CS + 2×GPIO), SX1262 (CS + BUSY + DIO1 + RST),
W25Q128 (CS). That is more than the S3 can spare through a 20‑pin header alone. Options
to resolve when we wire it up (flagged now so placement leaves room):

- a small **CS decoder / GPIO expander** (e.g. 74HC138 or an I²C expander) placed on the
  SPI spine, or
- letting the **C5/RV1106 own a subset** of the radios directly, or
- muxing IRQs.

Placement already reserves spine‑adjacent space for an expander so this doesn't force a
re‑floorplan later.

---

## Stackup (4‑layer)

| Layer | Role |
|---|---|
| `F.Cu` | signal (short local routing within each zone) |
| `In1.Cu` | **GND plane** — unbroken reference under every RF part |
| `In2.Cu` | **3V3 plane** |
| `B.Cu` | signal (back‑side modules + spine returns) |

Kept from v1 — dedicated GND/3V3 planes are the single biggest win for multi‑radio RF.
The plane names are set in each `.kicad_pcb` (`In1.Cu → "GND"`, `In2.Cu → "3V3"`).

---

## What is / isn't done in this pass

**Done (this "start off" pass):**
- Two openable KiCad projects (main‑board, shield) with the compact outline, 4‑layer
  stackup, mounting holes, and the shared J1 datum placed identically on both boards.
- Hierarchical schematics partitioned **one sheet per subsystem** — each sheet names its
  parts, its nets, and its board zone.
- Labelled placement **zones** drawn on the board (`User.1` front / `User.2` back) and a
  matching SVG floorplan per board.

**Next (follow‑on passes):**
1. Drop real symbols into each schematic sheet and wire the buses.
2. Assign the final ESP32‑S3 GPIO ↔ J1 pinout + resolve the CS/IRQ budget.
3. Place real footprints inside each zone, add the ground‑plane pours and stitching.
4. Hand‑route the RF nets (matched, short) then the spine, then everything else.
