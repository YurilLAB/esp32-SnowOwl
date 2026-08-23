# 🦉 SnowOwl — PCB

All PCB CAD lives here, with the **old** and **new** designs kept strictly separate.

```
PCB/
├── v1-original/          ← the ORIGINAL SnowOwl board (archived, read-only)
│   └── renders/          ← front/back renders (v1 was never published as CAD source)
└── v2-compact/           ← the NEW from-scratch 2-board redesign (active)
    ├── PLACEMENT.md      ← floorplan & bus strategy — read this first
    ├── main-board/       ← ESP32-S3 brain + UI  (SnowOwl-Main.kicad_pro)
    └── shield/           ← C5 + RV1106 + radios/readers (SnowOwl-Shield.kicad_pro)
```

## v1-original
The board shown in the top-level README renders. Only PNG renders exist — the original
KiCad source was never committed to this repo — so `v1-original/` archives those renders
as the record of what we're replacing. Nothing here is edited.

## v2-compact (active)
A **ground-up remake of both boards** as a stackable 2-board set, deliberately
floorplanned so it routes cleanly:

- **`main-board/`** — ESP32-S3 controller, TFT/UI, power, microSD, the stack header.
- **`shield/`** — ESP32-C5 (5 GHz), RV1106 ARM Linux, the Sub-GHz trio (CC1101/CC1200/
  SX1262), 3× nRF24, PN532 NFC, HTRC110 125 kHz, GPS, IR, iButton, W25Q128 flash.

Every radio/reader from v1 is kept; the size win comes from tight zoning and moving big
modules to the back. See **[`v2-compact/PLACEMENT.md`](v2-compact/PLACEMENT.md)** for the
zone map, the SPI-spine bus plan, and the J1 pin-grouping that keeps traces local.

> **Status:** floorplan skeleton. The projects open in KiCad 7–10 with the compact
> outline, 4-layer stackup, mounting holes, the shared J1 datum, and labelled placement
> zones. Symbols/footprints and routing come next. `*-floorplan.svg` in each board folder
> is a quick visual of its zones.

Made in KiCad. Fabrication outputs (`gerbers-out/`, zips) are git-ignored.
