# D03 — roaster control PCBs

Two PCBs for a 3 kg LPG drum roaster. This repo is the board design: schematic, KiCad, BOM, and fab notes. The machine firmware lives elsewhere. The later gas machine in this series is [Orion 05](https://github.com/byhabibi/orion-05). The electric sibling, including its firmware, is [D01E](https://github.com/byhabibi/d01e-roaster).

## Machine the boards serve

| Item | Value |
|---|---|
| Fuel | LPG, two solenoids |
| Power | 220 V, single phase, 50 Hz |
| Batch | 3 kg |
| HMI | Nextion NX8048K070 |
| Drum | VFD |
| Burner | servo on the gas valve |
| Thermocouples | three MAX31856 breakouts |

## Boards

| Board | KiCad | Role |
|---|---|---|
| PCB-01 MAIN-CTRL | `KiCad_LIVE_PRE_ROUTE/D03-MAIN-CTRL/` | ESP32-DevKitC seat, fixed 5 V buck, three thermocouple seats, HMI and logic |
| PCB-02 IO-PWR | `KiCad_LIVE_PRE_ROUTE/D03-IO-PWR/` | 24 V contactors, gas path, power distribution |

Open `D03-MAIN-CTRL.kicad_pro` and `D03-IO-PWR.kicad_pro`. Custom footprints are in each project's `D03.pretty` (DevKit, MP1584 module, MAX31856 breakout, contactor, SSR, fuse, PSU).

Drawings for review, without KiCad, are in `schematics/`. Start at `schematics/INDEX.html`. BOM lock is `BOM/D03_BOM_Production.csv`.

Target stack once a clean Gerber exists: 2-layer FR4, 1.6 mm, 1 oz copper, green mask, white silk.

## Do not fabricate the Gerbers in this snapshot

`PCB-01_MAIN-CTRL/Gerbers/` and `PCB-02_IO-PWR/Gerbers/` are placeholders. Both folders still carry a do-not-fab note. The KiCad projects are the pre-route seat lock: modules are placed and nets match the BOM, and the copper still has to be finished and pass DRC before a new Gerber export.

Seat lock used here:

- U1 = ESP32-DevKitC module, USB toward the top
- U3 = MP1584 fixed 5 V module, not a trimpot buck
- U4, U5, U6 = MAX31856 breakout modules
- Onboard USB-UART and its connector are do-not-place. Flash through the DevKit USB port
