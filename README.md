# Multi-Rail Power Management Board

LTspice and KiCad portfolio project for a sequenced three-rail power-management block.

## Project Summary

This project converts a single 5 V input into three sequenced rails for an SoC + sensor cluster:

| Rail | Voltage | Purpose | Enable Time |
| --- | ---: | --- | ---: |
| Rail A | 1.2 V | SoC core | 0 ms |
| Rail B | 1.8 V | I/O logic | 2 ms |
| Rail C | 3.3 V | Sensors/camera | 4 ms |

The design includes input protection, regulator-stage modeling, output capacitance, equivalent loads, enable sequencing, Rail C load-step simulation, KiCad schematic capture, BOM files, and interview/report documentation.

## Repository Layout

```text
ltspice/   LTspice schematics, netlist, symbols, and vendor model files
kicad/     KiCad schematic project, symbol library, ERC report, exports
docs/      complete report, interview Q&A, portfolio summary, run guide
results/   simulation plots
bom/       project BOM
```

## Main Files

- `ltspice/PowerRails_Behavioral_Complete.asc`
- `ltspice/PowerRails_Behavioral_Complete.cir`
- `kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_pro`
- `kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_sch`
- `docs/Exhaustive_Project_Report.md`
- `docs/Comprehensive_Interview_QA.md`
- `docs/Complete_Project_Report.md`
- `docs/Interview_Questions_and_Answers.md`
- `results/PowerRails_Sequencing_Plot.png`
- `results/RailC_Load_Transient_Plot.png`

## Validation

KiCad ERC result:

```text
0 errors, 0 warnings
```

The LTspice behavioral simulation is the recommended starting point because it does not depend on fragile vendor-symbol setup. Plot:

```text
V(RAIL_A_1V2)
V(RAIL_B_1V8)
V(RAIL_C_3V3)
V(EN_A)
V(EN_B)
V(EN_C)
```

## Engineering Note

The regulator part numbers are used for modeling and schematic practice. Before PCB fabrication, final regulator current ratings, thermal limits, package choices, and datasheet application circuits must be rechecked against the actual load requirements.
