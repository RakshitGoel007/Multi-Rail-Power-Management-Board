# Multi-Rail Power Management Board for SoC + Sensor Cluster

Prepared for: Rakshit Goel  
Project type: Hardware interview portfolio project  
Main tools: LTspice, KiCad, TI regulator models/datasheets

## 1. Executive Summary

This project designs a three-rail power management block from a single 5 V input. The target load is a small SoC plus sensor cluster needing a clean 1.2 V core rail, 1.8 V I/O rail, and 3.3 V sensor rail. The design focuses on correct power sequencing, load decoupling, test-point visibility, and a clear bring-up/debug story.

The key design rule is:

- Rail A, 1.2 V core, turns on first at 0 ms.
- Rail B, 1.8 V I/O, turns on second at about 2 ms.
- Rail C, 3.3 V sensors/camera, turns on third at about 4 ms.
- Power-down should happen in reverse order: 3.3 V off first, 1.2 V off last.

This sequencing avoids powering I/O pads before the core logic is alive, which reduces the risk of latch-up, back-powering, and undefined boot states.

## 2. Requirements

| Rail | Voltage | Max Current | Load Powered | Regulator Type | Enable Time |
| --- | ---: | ---: | --- | --- | ---: |
| Input | 5 V | 2 A | Board input | USB/input source | Always on |
| Rail A | 1.2 V | 500 mA | SoC core | Low-noise LDO | 0 ms |
| Rail B | 1.8 V | 600 mA | I/O logic, IMU | Synchronous buck | 2 ms |
| Rail C | 3.3 V | 800 mA | Sensors, camera | Synchronous buck | 4 ms |

Protection and bring-up requirements:

- Reverse-polarity protection on the 5 V input.
- Load switch on the 3.3 V sensor rail.
- Test points on VIN, every output rail, and every enable net.
- Output capacitors on all rails.
- Simulated sequencing and load transient response before schematic capture.

## 3. Architecture

The board accepts 5 V from USB or a bench supply. The 5 V input feeds three regulators:

- `U3 TLV70012`: creates the 1.2 V core rail.
- `U2 TPS62240`: creates the 1.8 V rail.
- `U1 TPS62203`: creates the 3.3 V rail.

The rail enable order is controlled by delayed enable signals:

- `EN_A`: immediate enable for 1.2 V.
- `EN_B`: delayed enable for 1.8 V.
- `EN_C`: delayed enable for 3.3 V.

In a real schematic, the delay can be made using RC networks feeding the regulator enable pins. For a production board, a dedicated power sequencer or supervisor IC would be more repeatable over voltage, temperature, and part tolerance.

## 4. Component Selection and BOM

| Ref | Part | Function | Package | Notes |
| --- | --- | --- | --- | --- |
| U1 | TI TPS62203 | 3.3 V synchronous buck | SOT23-5 | Used as guide-selected buck model |
| U2 | TI TPS62240 | 1.8 V synchronous buck | SOT23-5 / WSON | Used as guide-selected buck model |
| U3 | TI TLV70012 | 1.2 V LDO | SOT23-5 / SC70 / WSON family | Used as guide-selected LDO model |
| Q1 | AO3401 | P-MOSFET load switch | SOT23-3 | Controls/hot-plug protects Rail C |
| D1 | SS34 | Reverse-polarity Schottky diode | SMA / DO-214 | Input protection |
| C_IN | 10 uF ceramic X7R | Input bulk/decoupling | 0805/1206 | Place near regulator inputs |
| C_OUT_A | 22 uF ceramic X7R | 1.2 V output capacitor | 0805/1206 | Simulation uses 22 uF |
| C_OUT_B | 22 uF ceramic X7R | 1.8 V output capacitor | 0805/1206 | Simulation uses 22 uF |
| C_OUT_C | 22 uF ceramic X7R | 3.3 V output capacitor | 0805/1206 | Simulation uses 22 uF |
| R_LOAD_A | 2.4 ohm | 1.2 V full-load equivalent | 0603/0805, power-rated | 1.2 V / 0.5 A |
| R_LOAD_B | 3.0 ohm | 1.8 V full-load equivalent | 0603/0805, power-rated | 1.8 V / 0.6 A |
| I_LOAD_C | PULSE current sink | 3.3 V load transient | Simulation element | 0.1 A to 0.8 A |

Important engineering caveat:

The guide-selected regulator part numbers are useful for model practice, but their current ratings must be checked before fabrication. TPS62203/TPS62240 are low-current buck parts, and TLV700 is a low-current LDO family. If the final board must truly supply 500 mA, 600 mA, and 800 mA continuously, choose higher-current regulator alternatives and re-run thermal and transient checks. This is a good interview point because it shows that the BOM was not accepted blindly.

## 5. Load Calculations

Equivalent load resistance is calculated using:

```text
R = V / I
```

| Rail | Voltage | Current | Equivalent Load |
| --- | ---: | ---: | ---: |
| Rail A | 1.2 V | 0.5 A | 2.4 ohm |
| Rail B | 1.8 V | 0.6 A | 3.0 ohm |
| Rail C | 3.3 V | 0.8 A | 4.125 ohm |

For the load-transient simulation, Rail C uses a current sink instead of a fixed 4.125 ohm resistor:

```spice
PULSE(0.1 0.8 8m 10u 10u 4m 8m)
```

This means the sensor rail draws 0.1 A at light load, then jumps to 0.8 A at 8 ms.

## 6. LTspice Simulation

The main simulation deliverables are:

- `PowerRails_Behavioral_Complete.asc`
- `PowerRails_Behavioral_Complete.cir`

The behavioral simulation is intentionally robust. It avoids depending on custom symbols or vendor-model compatibility and instead represents each regulator as an ideal sequenced voltage source with small output resistance and output capacitance.

Run command:

```spice
.tran 20m startup
```

Signals to plot:

```text
V(RAIL_A_1V2)
V(RAIL_B_1V8)
V(RAIL_C_3V3)
V(EN_A)
V(EN_B)
V(EN_C)
I(IC_LOADSTEP)
```

Expected sequencing result:

- `RAIL_A_1V2` rises first and settles near 1.2 V.
- `RAIL_B_1V8` starts after 2 ms and settles near 1.8 V.
- `RAIL_C_3V3` starts after 4 ms and settles near 3.3 V.
- At 8 ms, Rail C load current steps from 0.1 A to 0.8 A.

The simulation includes these measurement commands:

```spice
.meas tran RailA_90pct_time WHEN V(RAIL_A_1V2)=1.08 RISE=1
.meas tran RailB_90pct_time WHEN V(RAIL_B_1V8)=1.62 RISE=1
.meas tran RailC_90pct_time WHEN V(RAIL_C_3V3)=2.97 RISE=1
.meas tran RailC_pre_step_avg AVG V(RAIL_C_3V3) FROM=7m TO=7.8m
.meas tran RailC_load_step_min MIN V(RAIL_C_3V3) FROM=8m TO=9m
.meas tran RailC_recovered_avg AVG V(RAIL_C_3V3) FROM=9m TO=12m
```

Acceptance criteria:

- Rail A must be above 90% before Rail B starts.
- Rail B must be above 90% before Rail C starts.
- Rail C transient droop should remain within about +/-3% of 3.3 V for this portfolio-level model.

## 7. KiCad Schematic

The KiCad deliverable is located in:

```text
PowerRailsBoard_KiCad/
```

Main files:

- `PowerRailsBoard.kicad_pro`: KiCad project file.
- `PowerRailsBoard.kicad_sch`: ERC-clean schematic.
- `PowerRailsBoard_schematic.pdf`: exported schematic PDF.
- `PowerRailsBoard.net`: exported netlist.
- `PowerRailsBoard_BOM_from_KiCad.csv`: BOM exported directly from KiCad.
- `ERC_report.txt`: KiCad ERC report showing 0 errors and 0 warnings.

The schematic contains:

- One 5 V input connector or power symbol.
- `D1 SS34` reverse-polarity protection at the input.
- `U3 TLV70012` or a corrected higher-current LDO for 1.2 V.
- `U2 TPS62240` or a corrected higher-current buck for 1.8 V.
- `U1 TPS62203` or a corrected higher-current buck for 3.3 V.
- `Q1 AO3401` P-MOSFET load switch on Rail C.
- RC enable-delay networks for `EN_A`, `EN_B`, and `EN_C`.
- 10 uF input capacitors and 22 uF output capacitors.
- Test points on `VIN_5V`, `EN_A`, `EN_B`, `EN_C`, `RAIL_A_1V2`, `RAIL_B_1V8`, and `RAIL_C_3V3`.
- Clear net labels and annotated reference designators.

Recommended KiCad net names:

```text
VIN_5V
EN_A
EN_B
EN_C
RAIL_A_1V2
RAIL_B_1V8
RAIL_C_3V3
GND
```

ERC target:

- Achieved: KiCad CLI ERC reports 0 errors and 0 warnings.
- Generic embedded symbols are used so the schematic opens without depending on missing external part symbols.
- Exact manufacturer footprints should be assigned before PCB layout/fabrication.

## 8. Protection and Bring-Up Strategy

Reverse-polarity protection:

- `SS34` protects against accidental reverse input.
- A Schottky diode is simple but drops voltage and dissipates power.
- For higher efficiency, an ideal-diode controller or P-MOSFET reverse-protection circuit is better.

Rail C load switch:

- `AO3401` P-MOSFET can disconnect the 3.3 V sensor cluster.
- This helps with hot-plug protection and allows firmware-controlled sensor power.

Test points:

- Test points make bring-up measurable.
- During bench validation, probe `VIN_5V`, all enable pins, and all output rails with an oscilloscope.

## 9. What To Say In An Interview

Use this short explanation:

This project converts a 5 V input into three sequenced rails for an SoC and sensor cluster. I chose a 1.2 V core rail, 1.8 V I/O rail, and 3.3 V sensor rail. The core rail comes up first, then I/O, then sensors, because powering I/O before the core can cause undefined logic states or latch-up. I verified the sequencing in LTspice and added a Rail C load-transient test to model a sensor waking up. For a fabricated version, I would update the regulator current ratings, run thermal checks, and validate the rails on the bench with a scope and electronic load.

## 10. Limitations and Next Steps

Current limitations:

- Behavioral LTspice model is used for the main proof, so it does not show real switching ripple.
- Vendor model integration is prepared, but real-model convergence must be checked separately.
- The guide-selected part numbers should be current-rating checked before fabrication.
- PCB layout is not fully routed; the current KiCad deliverable is an ERC-clean schematic-level design.

Next steps:

- Replace guide-selected parts with regulators rated for the required currents.
- Assign exact footprints in KiCad.
- Do a rough 2-layer placement pass.
- For a real board, prefer a 4-layer PCB with solid ground planes for better return paths and lower noise.
- Fabricate and validate sequencing, ripple, load transient, and temperature rise on the bench.

## 11. Project Files

| File | Purpose |
| --- | --- |
| `PowerRails_Behavioral_Complete.asc` | Main LTspice schematic-style simulation, no custom symbols required |
| `PowerRails_Behavioral_Complete.cir` | Same simulation as a plain LTspice netlist |
| `PowerRails_REAL_clean.asc` | Starter schematic using custom regulator symbols |
| `PowerRailsBoard_KiCad/PowerRailsBoard.kicad_pro` | KiCad project |
| `PowerRailsBoard_KiCad/PowerRailsBoard.kicad_sch` | ERC-clean KiCad schematic |
| `PowerRailsBoard_KiCad/PowerRailsBoard_schematic.pdf` | Exported KiCad schematic PDF |
| `PowerRailsBoard_KiCad/ERC_report.txt` | KiCad ERC report, 0 errors and 0 warnings |
| `PowerRailsBoard_KiCad/PowerRailsBoard.net` | Exported KiCad netlist |
| `PowerRailsBoard_KiCad/PowerRailsBoard_BOM_from_KiCad.csv` | BOM exported from KiCad |
| `TPS62203.asy` | LTspice symbol for TPS62203 subcircuit |
| `TPS62240_TRANS.asy` | LTspice symbol for TPS62240 subcircuit |
| `TLV70012.asy` | LTspice symbol for TLV70012 subcircuit |
| `TI_regulator_models/` | Downloaded TI model ZIPs, extracted libraries, and datasheets |
| `Complete_Project_Report.md` | This report |
| `Interview_Questions_and_Answers.md` | Interview question bank |
| `PowerRails_BOM.csv` | BOM table for project documentation |
