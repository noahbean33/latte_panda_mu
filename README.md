# LattePanda Mu Carrier Board (KiCad)

A custom carrier-board reference design for the **LattePanda Mu** compute module, used as a worked example in the Tech Explorations KiCad course:
<https://connect.techexplorations.com/so/high-speed-pcb-design-with-kicad>

This repository contains the KiCad 7+ source, project libraries, generated Gerbers, and assembly outputs for a 4-layer carrier board that breaks out the LattePanda Mu module to real-world I/O: Ethernet, HDMI, USB 2.0/3.0, PCIe x4, an M.2 E-key slot, GPIO, fan control, and a full power supply.

The design is provided as **reference material** for students following the course.

---

## What is on the board

| Block | Sheet | Notes |
|---|---|---|
| Compute module | `LattePanda_Module.kicad_sch` | LattePanda Mu module carrier interface |
| Power supply | `psu.kicad_sch` | Multi-rail power for the module and peripherals |
| Ethernet | `ethernet.kicad_sch` | RJ45 + magnetics |
| HDMI | `hdmi.kicad_sch` | HDMI output |
| USB 2.0 / 3.0 | `usb_2_3.kicad_sch` | Host/device ports |
| PCIe x4 | `pciex4.kicad_sch` | PCIe x4 slot |
| M.2 E-key | `m2_em_key.kicad_sch` | M.2 slot for wireless / accelerator cards |
| GPIO | `gpio.kicad_sch` | General-purpose I/O breakout |
| Fan control | `fan.kicad_sch` | On-board fan header + control |

The full BOM is in [`LattePandaMu_carrier_custom.csv`](LattePandaMu_carrier_custom.csv).

---

## Repository layout

```
.
├── LattePandaMu_carrier_custom.kicad_pro   # KiCad project — open this in KiCad
├── LattePandaMu_carrier_custom.kicad_pcb   # 4-layer PCB layout
├── LattePandaMu_carrier_custom.kicad_sch   # Root (hierarchical) schematic
├── LattePandaMu_carrier_custom.kicad_dru   # Custom design rules
├── LattePandaMu_carrier_custom.csv         # BOM export
├── *.kicad_sch                             # Hierarchical sub-sheets (see table above)
├── sym-lib-table                           # Project-scoped symbol library table
├── fp-lib-table                            # Project-scoped footprint library table
├── Libraries/                              # Custom symbols, footprints, 3D models
├── Assembly/                               # Pick-and-place files (top / bottom)
├── LattePandaMu_v1_gerbers/                # Gerbers for the v1 fabrication run
└── LattePandaMu_v1.1_gerbers/              # Gerbers for the v1.1 fabrication run
```

The `sym-lib-table` and `fp-lib-table` use `${KIPRJMOD}` so the project-local `Libraries/` folder is picked up automatically — no global library installation required.

---

## Getting started

1. Install **KiCad 7 or newer** (<https://www.kicad.org/download/>).
2. Clone this repository:
   ```bash
   git clone https://github.com/futureshocked/kicad_latte_panda_mu.git
   ```
3. Open `LattePandaMu_carrier_custom.kicad_pro` in KiCad.
4. Explore the schematic hierarchy starting from `LattePandaMu_carrier_custom.kicad_sch` (double-click sheet symbols to descend into sub-sheets).
5. Open the PCB editor to view the 4-layer layout, or browse the ready-made Gerbers in `LattePandaMu_v1.1_gerbers/`.

---

## What is intentionally NOT in this repository

To keep the checkout small and focused on the design, the following are excluded:

- KiCad autosave files and the footprint info cache (`fp-info-cache`)
- KiCad per-user editor state (`*.kicad_prl`)
- The `LattePandaMu_carrier_custom-backups/` folder of KiCad-generated zip snapshots
- The `.history/` folder from the VS Code *Local History* extension
- Redundant top-level `.zip` snapshots of the project, Assembly, and Gerbers folders

These are regenerated on demand by KiCad and the editor.

---

## License and use

This project is provided as course material for Tech Explorations students. You are welcome to use it for learning, experimentation, and personal projects. Please do not redistribute it as your own work.

For questions or discussion, please use the course community on Tech Explorations Connect.
