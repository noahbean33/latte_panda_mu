# LattePanda Mu Carrier Board

A custom 4-layer carrier board for the **LattePanda Mu** x86 compute module (Intel N100 / N305), designed in KiCad. The module supplies the CPU, RAM and storage; this board turns its 260-pin edge connector into real-world I/O — Gigabit Ethernet, HDMI, USB 2.0/3.0, two M.2 slots, GPIO/I²C/UART headers, PWM fan control, and a dual-input power supply.

![Top view of the carrier board](docs/board-top.png)

*3D render from the current KiCad source. Most connectors have no 3D model attached, so they appear as bare pads.*

---

## Board at a glance

| | |
|---|---|
| **Outline** | 170.1 × 100.1 mm, 8 mounting holes (M3 + M2), 2 fiducials |
| **Stackup** | 4 layers — `F.Cu` / `In1.GND` / `In2.PWR` / `B.Cu`, 1.606 mm, FR4 εr 4.5, impedance-controlled |
| **Power in** | 12 V barrel jack **or** USB-C PD (CH224K negotiates 15 V) — diode-ORed onto a common `VDC` rail |
| **Rails** | `VDC` → 2× LM2734X buck → **+5V**, **+3V3**; **+1V0** local to the Ethernet PHY; **+BATT** from a CR1220 for the module RTC |
| **Ethernet** | RTL8111H-CG GbE controller on a PCIe x1 link, RJ45 with integrated magnetics |
| **Display** | HDMI Type-A, driven from the module's `DDI_B` port |
| **USB** | 2× USB 3.0 Type-A (stacked), 2× USB 2.0 Type-A (stacked), 1× USB-C (**power only**) |
| **M.2** | M-key (PCIe x1, NVMe) + E-key (PCIe x1 + USB 2.0, Wi-Fi/BT) |
| **Expansion** | PCIe x4 slot — fully designed, **not populated** in this revision |
| **Misc** | 4-pin PWM/tach fan header, power + reset buttons, debug UART, 13 test points (10 populated) |
| **Size of the job** | 217 placed footprints, 354 nets, 680 vias, 9 hierarchical sheets |

Full parts list: [`LattePandaMu_carrier_custom.csv`](LattePandaMu_carrier_custom.csv).

---

## Where the high-speed lanes go

The Mu exposes a *fixed* pool of high-speed I/O (HSIO) lanes. Every interface on the board is spent out of that pool — this table is the single most important design decision in the project:

| Module resource | Spent on | Notes |
|---|---|---|
| `HSIO_0` TX/RX | USB 3.0 port 1 SuperSpeed | TX AC-coupled (C26/C27) |
| `HSIO_1` TX/RX | USB 3.0 port 2 SuperSpeed | TX AC-coupled (C28/C29) |
| `HSIO_2` + `REFCLK0` | M.2 **M**-key — PCIe x1 | TX AC-coupled (C49/C50) |
| `HSIO_3` + `REFCLK3` | M.2 **E**-key — PCIe x1 | TX AC-coupled (C55/C56) |
| Dedicated GbE PCIe pair + clock | RTL8111H | AC-coupled (C38/C39/C42/C43) |
| `HSIO_8` – `HSIO_11` | *reserved* for the PCIe x4 slot | currently unrouted (slot is DNP) |
| `DDI_B_TX0..3` | HDMI TMDS | AC-coupled, ESD arrays U1/U2 |
| `USB2_P1/2/3/5` | 4× USB Type-A ports | |
| `USB2_P7` | M.2 E-key | Bluetooth on a combo card |
| `USB2_P4/6/8` | unused | |

AC coupling appears on **TX pairs only** — the far-end receiver provides its own caps, per PCIe and USB 3.0 convention.

---

## Schematic hierarchy

| Sheet | File | Contents |
|---|---|---|
| LattePanda Module | [`LattePanda_Module.kicad_sch`](LattePanda_Module.kicad_sch) | 260-pin module connector and all lane fan-out |
| Power Supply | [`psu.kicad_sch`](psu.kicad_sch) | DC jack, USB-C PD (CH224K), ORing, 2× buck, power sequencing |
| Gigabit Ethernet | [`ethernet.kicad_sch`](ethernet.kicad_sch) | RTL8111H, 25 MHz crystal, MDI pairs, RJ45 |
| HDMI | [`hdmi.kicad_sch`](hdmi.kicad_sch) | DDI_B → TMDS, ESD protection, fused +5 V |
| USB 2.0 & 3.0 | [`usb_2_3.kicad_sch`](usb_2_3.kicad_sch) | Stacked A-connectors, per-port fusing, ESD |
| M.2 M Key | [`m2_em_key.kicad_sch`](m2_em_key.kicad_sch) | Both the M-key and E-key sockets |
| PCIe x4 | [`pciex4.kicad_sch`](pciex4.kicad_sch) | x4 slot + AC caps — all DNP |
| GPIO | [`gpio.kicad_sch`](gpio.kicad_sch) | 4× I²C, 3× UART, 4× GPP headers |
| Fan | [`fan.kicad_sch`](fan.kicad_sch) | 4-pin PWM fan header and drive |

---

## Design rules

Net classes are pattern-assigned in the project file, so a net gets the right geometry from its name:

| Class | Applies to | Track / diff-pair geometry |
|---|---|---|
| `100Ω` | `*MDI*` (Ethernet magnetics side) | 0.144 mm width / 0.100 mm gap |
| `90Ω` | USB 2.0/3.0, PCIe, HSIO, TMDS, REFCLK | 0.199 mm width / 0.099 mm gap |
| `PWR` | `*+*V*` rails | 0.3 mm track, 0.2 mm clearance |
| `Default` | everything else | 0.15 mm track |

Board minimums: 0.14 mm track, 0.2 mm via / 0.2 mm hole, 0.1 mm annular ring, 0.2 mm copper-to-edge.

> [`LattePandaMu_carrier_custom.kicad_dru`](LattePandaMu_carrier_custom.kicad_dru) contains intra-pair **skew rules that are still commented out**. Length matching is therefore not currently enforced by DRC.

---

## Repository layout

```
.
├── LattePandaMu_carrier_custom.kicad_pro   # open this in KiCad
├── LattePandaMu_carrier_custom.kicad_pcb   # 4-layer layout
├── LattePandaMu_carrier_custom.kicad_sch   # root hierarchical schematic
├── LattePandaMu_carrier_custom.kicad_dru   # custom design rules
├── LattePandaMu_carrier_custom.csv         # BOM export
├── *.kicad_sch                             # sub-sheets (table above)
├── sym-lib-table / fp-lib-table            # project-scoped, ${KIPRJMOD}-relative
├── Libraries/                              # custom symbols and footprints
├── Assembly/                               # pick-and-place (top / bottom)
├── docs/                                   # renders and documentation images
├── .copperpilot/                           # automated design-review output
├── LattePanda_reference/                   # DFRobot's official Mu resources (MIT)
├── LattePandaMu_v1_gerbers/                # historical fab output — see caveat below
└── LattePandaMu_v1.1_gerbers/              # historical fab output — see caveat below
```

---

## Getting started

1. Install **KiCad 9 or newer** (the project is saved by KiCad 10) — <https://www.kicad.org/download/>.
2. Clone the repository:
   ```bash
   git clone https://github.com/noahbean33/latte_panda_mu.git
   ```
3. Open `LattePandaMu_carrier_custom.kicad_pro`. The library tables are `${KIPRJMOD}`-relative, so `Libraries/` is picked up with no global install.
4. Descend into sub-sheets from the root schematic; open the PCB editor for the layout.

Re-run the checks yourself:

```bash
kicad-cli sch erc --format json -o erc.json LattePandaMu_carrier_custom.kicad_sch
```

```bash
kicad-cli pcb drc --format json -o drc.json LattePandaMu_carrier_custom.kicad_pcb
```

---

## Current status

Routing is **complete** — DRC reports **0 unconnected items** and **0 schematic-parity errors** across all 354 nets. What remains is cleanup.

| Check | Result |
|---|---|
| DRC unconnected / parity | 0 |
| DRC clearance errors | 20 — teardrops encroaching on `+3V3` / `VDC` / `GND` zone clearance |
| DRC courtyard overlaps | 4 — module `A1` vs `H5`, `H6`, `R78`, `R79` |
| DRC silkscreen warnings | 13 — clipped by board edge or overlapping mask |
| ERC errors | 4 dangling hierarchical labels (`HSIO_10_RX±`, `HSIO_11_RX±`) |
| ERC warnings | 21 library-symbol mismatches, 13 redundant net names |

**Known issues, in priority order**

1. **20 teardrop clearance errors.** Teardrops on `GND` sit ~0.141 mm from the `+3V3` zone where the `PWR` class demands 0.200 mm. Either shrink the teardrops or adjust clearance in those regions.
2. **4 courtyard overlaps** where the module footprint `A1` covers two M2 mounting holes and two resistors. This needs a mechanical decision: is the module courtyard oversized, or is the placement genuinely wrong?
3. **4 dangling labels** left over from dropping the PCIe x4 slot — `HSIO_10/11_RX±` still terminate in mid-air on the root sheet.
4. **CH224K (U14) `CFG2`/`CFG3`/`PG` float.** These strap the negotiated PD voltage; confirm the floating default matches the datasheet's intended profile before building.
5. **Library drift.** 21 symbols and 5 footprints were edited in-project and no longer match `A_HDJ_Library`, whose original metadata still points at an absolute path on the vendor's machine.
6. **The committed gerbers are stale.** Both sets were plotted from KiCad 9 in September and November 2025 and predate the routing rework — they do **not** describe the board in this repository. Re-plot before ordering.

---

## What I learned building this

**Carrier-board design is lane budgeting, not circuit invention.** Almost nothing here is a novel circuit; the module already contains the hard parts. The real work was deciding how to spend a fixed pool of HSIO lanes and then honouring the impedance, coupling and reference-clock requirements of each one. The lane table above *is* the design.

**Removing a block is a whole-schematic edit.** Dropping the PCIe x4 slot looked like one deleted connector, and it left four hierarchical labels dangling at the root, four HSIO pairs stubbed at the module, and 16 AC caps orphaned in the BOM. ERC caught it; reading the schematic myself did not.

**"DNP / excluded from board" is a real design tool.** Keeping the x4 slot, its caps and its test point in the schematic while excluding them from the board preserves a documented, one-edit-away option instead of burying the work in git history. The price is a BOM full of DNP lines and a permanent stream of ERC noise — worth accepting consciously rather than ignoring.

**A reference design is a starting point, not a shortcut.** This began from DFRobot's DFR1142 Lite Carrier (146 × 102 mm) and ended at 170.1 × 100.1 mm with a different sheet structure. Once the outline moves, none of the reference routing survives. What actually carried over was the *schematic decisions*, not the layout.

**Post-processing features have to be run against the same rules as routing.** All 20 remaining clearance errors come from teardrops added after routing was finished, colliding with the 0.2 mm `PWR` clearance. Turning on a global feature at the end of a project produces errors in bulk; turning it on early produces them one at a time, where each is cheap to fix.

**Impedance control is geometry, not a checkbox.** The `90Ω` and `100Ω` track widths are only correct for this specific stackup — 0.21 mm prepreg, εr 4.5, 1.065 mm core. Letting the fab substitute its house stackup would silently invalidate every high-speed trace on the board, which is why the stackup is committed rather than left to a default.

**Courtyards are mechanical claims, and DRC treats them as absolute.** The overlaps between the module and the M2 standoffs aren't electrical faults — they're an assertion about physical clearance under the module. DRC can flag the conflict but can't say which side is wrong; that takes the mechanical drawing.

**Vendor libraries have to be vendored on day one.** The `A_HDJ_Library` symbols still carry an absolute path from the original author's external drive. Because the project-scoped library tables use `${KIPRJMOD}` the project opens fine, but 26 mismatch warnings now sit permanently between me and a clean ERC/DRC run — and they would bite anyone cloning this on another machine.

**Automated review finds the boring problems; a person still has to read the datasheet.** ERC and DRC found library drift, silkscreen clipping and clearance violations. Neither could tell me that the CH224K's floating `CFG` pins select a PD voltage profile — that came from tracing the netlist by hand against the datasheet.

**Committed build outputs go stale silently.** Two sets of gerbers sat in this repository looking authoritative while the board underneath them changed completely. Fab outputs belong to a tagged release or a CI job, not to a working tree.

**Scope discipline is what made this finishable.** The project started as an AMC RTM carrier for MicroTCA chassis — dual NICs, two USB hubs, JTAG debug, an ARM management controller. What shipped is a single-PHY board derived from the Lite Carrier. Cutting the ambitious version is the decision that produced a routed board instead of a half-finished schematic.

---

## Next steps

- [ ] Resolve the 20 teardrop clearance errors and 4 courtyard overlaps
- [ ] Tie off or delete the dangling `HSIO_10/11_RX±` labels
- [ ] Confirm CH224K CFG strapping against the datasheet
- [ ] Re-sync symbols and footprints to `Libraries/` and fix the stale absolute path
- [ ] Uncomment and tune the intra-pair skew rules in the `.kicad_dru`
- [ ] Re-plot gerbers and pick-and-place from the current board, tagged as v2

---

## Credits and licensing

- [`LattePanda_reference/`](LattePanda_reference/) is DFRobot's official LattePanda Mu development resource pack — pinouts, mechanical drawings, symbol/footprint libraries, and the DFR1141 / DFR1142 / DFR1144 reference carriers. It carries its own [MIT license](LattePanda_reference/LICENSE) and is included unmodified.
- The schematic structure and much of the power and I/O design derive from the DFR1142 Lite Carrier reference and from the `futureshocked/kicad_latte_panda_mu` course project.
- No license file currently applies to the custom design at the repository root.
