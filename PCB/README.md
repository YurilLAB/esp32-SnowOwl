# 🦉 SnowOwl — PCB

**Clean slate.** All previous CAD — boards, schematics, footprints, floorplan and renders — has
been wiped. The hardware direction is changing and the design is being started over from scratch.

```
PCB/
└── RESEARCH.md    ← component & schematic reference carried over from the old attempt
```

## RESEARCH.md

The only thing kept: the electrical research worth not repeating.

- **System architecture** — which processor owns what, and the SPI-spine approach.
- **Component inventory** — every radio/reader, and which ones are self-contained modules vs.
  chip-down parts that need a crystal and an RF front-end.
- **Power** — the single-USB-C / battery topology, plus **datasheet pinout corrections** for the
  charger, regulators and fuel gauge (the old symbols had invented pinouts).
- **RF front-end topologies** — matching + filter chains for the chip-down radios, with the
  caveat that every L/C value still needs VNA tuning.
- **Schematic gotchas** — including the net-merge bug that silently shorted whole nets twice.

It deliberately assumes **no particular board size, layout or form factor**, so none of it
constrains the new design.

> Fabrication outputs (`gerbers-out/`, zips) are git-ignored.
