# Exhaustive Project Report: Multi-Rail Power Management Board

Prepared for: Rakshit Goel  
Project type: hardware design, power electronics, schematic capture, simulation, and interview portfolio  
Main tools: LTspice, KiCad, TI regulator models, datasheets, behavioral simulation  
Repository: `Multi-Rail-Power-Management-Board`

## 1. Project Overview

This project is a multi-rail power management design for a small embedded system made from an SoC and a sensor/camera cluster. The board takes a single 5 V input and generates three regulated voltage rails:

| Rail | Voltage | Intended Load | Enable Order |
| --- | ---: | --- | ---: |
| Rail A | 1.2 V | SoC core logic | First |
| Rail B | 1.8 V | I/O logic, memory interface, IMU-style devices | Second |
| Rail C | 3.3 V | sensors, camera, peripheral modules | Third |

The core idea is not just to generate voltages. The project demonstrates an engineering workflow:

- define voltage/current requirements,
- choose a power architecture,
- model startup sequencing,
- simulate load transient behavior,
- capture a readable KiCad schematic,
- include protection and test points,
- prepare documentation and interview explanation material.

The project is portfolio-oriented. It is designed to show that the designer understands both circuit behavior and the practical workflow of building and explaining a hardware subsystem.

## 2. What The Project Does

At a system level, the project does the following:

1. Accepts a nominal 5 V input supply.
2. Provides reverse-polarity protection at the input.
3. Generates a 1.2 V rail for SoC core logic.
4. Generates a 1.8 V rail for I/O logic and low-voltage peripherals.
5. Generates a 3.3 V rail for sensors/camera/peripherals.
6. Sequences the rails in a controlled order.
7. Adds output capacitors to support transient current demand.
8. Adds equivalent loads for simulation and validation.
9. Adds a Rail C load-step event to represent a sensor/camera waking up.
10. Adds enable monitor signals so startup timing can be plotted clearly.
11. Provides a KiCad schematic with test points and a BOM.
12. Exports a schematic PDF, SVG/PNG preview, netlist, BOM, and ERC report.
13. Provides a complete report and interview question bank.

The LTspice simulation focuses on startup behavior and load transient response. The KiCad schematic focuses on readability, design intent, component organization, and bring-up/debug access.

## 3. Why Multi-Rail Power Exists

Modern embedded systems rarely run from one supply voltage. A typical SoC may have:

- a low-voltage core supply for internal logic,
- a separate I/O supply for interface pins,
- a higher peripheral supply for sensors, cameras, RF modules, or analog front ends.

Using separate rails allows each subsystem to receive the voltage it needs. It also improves control over noise, sequencing, power consumption, and debugging.

In this project:

- 1.2 V represents a core rail.
- 1.8 V represents an I/O rail.
- 3.3 V represents a peripheral/sensor rail.

This is a realistic split for many embedded boards.

## 4. Design Requirements

### 4.1 Electrical Requirements

| Requirement | Target |
| --- | --- |
| Input voltage | 5 V nominal |
| Output Rail A | 1.2 V |
| Output Rail B | 1.8 V |
| Output Rail C | 3.3 V |
| Rail A current target | 500 mA equivalent |
| Rail B current target | 600 mA equivalent |
| Rail C current target | 800 mA transient case |
| Startup sequence | A first, B second, C third |
| Rail B delay | about 2 ms after Rail A |
| Rail C delay | about 4 ms after Rail A |
| Rail C load step | 0.1 A to 0.8 A at 8 ms |
| Simulation duration | 20 ms transient |

### 4.2 Functional Requirements

The project must:

- show clear startup sequencing,
- include realistic rail names,
- include output capacitance,
- model expected load conditions,
- include transient response observation,
- include test points,
- be explainable in an interview,
- open reliably in LTspice and KiCad.

### 4.3 Documentation Requirements

The project must include:

- a technical report,
- an interview Q&A bank,
- a one-page summary,
- instructions for opening and running,
- BOM files,
- simulation plots,
- schematic exports.

## 5. Power Architecture

The architecture is:

```text
5 V input
  |
  +--> input protection
  |
  +--> Rail A regulator --> 1.2 V core rail
  |
  +--> Rail B regulator --> 1.8 V I/O rail
  |
  +--> Rail C regulator/load switch --> 3.3 V sensor rail
```

Rail A comes up first because SoC internal logic should be alive before I/O and peripherals are powered. Rail B comes second because I/O pads often depend on the core domain being valid. Rail C comes third because sensors/cameras can draw significant current and should not be powered before the processor/I/O side is ready.

## 6. Rail Definitions

### 6.1 Rail A: 1.2 V Core

Rail A is the first rail. It represents the SoC core supply. Core rails often power CPU logic, internal state machines, memory controllers, PLL blocks, or digital logic.

Design intent:

- start at 0 ms,
- settle near 1.2 V,
- support an equivalent 500 mA load,
- provide a reference for later rails.

Equivalent load:

```text
R = V / I = 1.2 V / 0.5 A = 2.4 ohm
```

### 6.2 Rail B: 1.8 V I/O

Rail B is the second rail. It represents low-voltage I/O logic or peripheral interfaces.

Design intent:

- start around 2 ms,
- settle near 1.8 V,
- support an equivalent 600 mA load,
- avoid powering I/O before the core rail.

Equivalent load:

```text
R = V / I = 1.8 V / 0.6 A = 3.0 ohm
```

### 6.3 Rail C: 3.3 V Sensor Rail

Rail C is the third rail. It represents sensors, camera modules, and other 3.3 V peripherals.

Design intent:

- start around 4 ms,
- settle near 3.3 V,
- support a transient load jump,
- include a load-switch concept for peripheral control.

Full-load equivalent resistance:

```text
R = V / I = 3.3 V / 0.8 A = 4.125 ohm
```

In the main simulation, Rail C uses a current load-step model because that better represents a sensor or camera suddenly waking up.

## 7. Why Sequencing Matters

Power sequencing is important because ICs are not always tolerant of arbitrary rail order. Incorrect sequencing can cause:

- back-powering through I/O pins,
- current injection through ESD diodes,
- undefined boot states,
- latch-up risk,
- peripherals starting before the controller is ready,
- increased inrush current,
- failed reset behavior.

The intended startup order is:

```text
Rail A: 1.2 V core       t = 0 ms
Rail B: 1.8 V I/O        t = 2 ms
Rail C: 3.3 V sensors    t = 4 ms
```

For power-down, the preferred order is normally reverse:

```text
Rail C off first
Rail B off second
Rail A off last
```

This prevents peripherals and I/O from remaining powered while the core logic is unavailable.

## 8. Component-Level Design

### 8.1 Input Protection

The schematic includes `D1 SS34`, a Schottky diode used for reverse-polarity protection. The purpose is to protect the board if the input supply is accidentally connected backwards.

Advantages:

- simple,
- cheap,
- easy to understand,
- robust for a student-level schematic.

Disadvantages:

- forward voltage drop,
- power loss,
- heating at high current.

Power loss estimate:

```text
P = I * Vf
```

If current is 1 A and diode drop is 0.4 V:

```text
P = 1 A * 0.4 V = 0.4 W
```

For a production board, a P-MOSFET ideal-diode style reverse-protection circuit would be more efficient.

### 8.2 Regulator Stages

The guide-selected regulators/models are:

| Ref | Part | Purpose |
| --- | --- | --- |
| U1 | TPS62203 | buck regulator reference for 3.3 V rail |
| U2 | TPS62240 | buck regulator reference for 1.8 V rail |
| U3 | TLV70012 | LDO reference for 1.2 V rail |

Important caveat:

The selected part numbers are useful for model practice and schematic learning, but a real fabricated board must verify regulator current ratings. If the actual rails require 500 mA, 600 mA, and 800 mA continuously, the selected parts may not all be suitable. A competent final design would select regulators with adequate current margin, thermal performance, and package capability.

### 8.3 Output Capacitors

Each rail uses a 22 uF output capacitor in the simulation. Output capacitors:

- reduce ripple,
- support transient load steps,
- improve local energy availability,
- affect regulator loop stability,
- reduce high-frequency impedance at the rail.

Ceramic X7R capacitors are used conceptually because they have good temperature behavior compared with lower-grade dielectrics.

Real-world note:

Ceramic capacitor effective capacitance decreases with DC bias. A 22 uF capacitor may behave like a much smaller capacitor at its operating voltage, depending on package and dielectric.

### 8.4 Equivalent Loads

Resistive loads are used for Rail A and Rail B:

| Rail | Voltage | Current | Load |
| --- | ---: | ---: | ---: |
| A | 1.2 V | 0.5 A | 2.4 ohm |
| B | 1.8 V | 0.6 A | 3.0 ohm |

Rail C includes a light load and a pulsed load step.

### 8.5 Rail C Load Step

Rail C models a sudden load increase:

```text
0.1 A light load -> 0.8 A active load
```

This represents a sensor/camera subsystem waking up or entering active mode.

The project measures:

- pre-step average voltage,
- minimum voltage during load step,
- recovered average voltage.

This shows whether the rail droops too much and whether it recovers.

### 8.6 Enable Monitor Signals

The LTspice schematic includes behavioral enable monitor sources:

```text
EN_A
EN_B
EN_C
```

These are not meant to be final production enable circuits. They are included so the startup sequence can be plotted clearly against the rail voltages.

In a real design, these enables could be produced by:

- RC delay circuits,
- power-good chaining,
- a microcontroller,
- a PMIC,
- a supervisor/sequencer IC.

## 9. LTspice Simulation

### 9.1 Main Files

```text
ltspice/PowerRails_Behavioral_Complete.asc
ltspice/PowerRails_Behavioral_Complete.cir
```

The `.asc` file is the visual schematic-style simulation. The `.cir` file is the companion plain netlist.

### 9.2 Why Behavioral Modeling Was Used

Vendor regulator models can be fragile. They may depend on exact subcircuit names, hidden pins, PSPICE compatibility, or symbol alignment. A behavioral model allows the project to validate the architecture first:

- startup order,
- expected output voltages,
- enable timing,
- load step behavior,
- measurement workflow.

The behavioral model is not a replacement for a final regulator simulation. It is a fast architecture-level verification step.

### 9.3 Behavioral Equations

The LTspice schematic uses function directives such as:

```spice
.func RA(t) if(t>=0,1.2,0)
.func RB(t) if(t>=2m,1.8,0)
.func RC(t) if(t>=4m,3.292,0)-if(t>=8m,if(t<12m,56m,0),0)
.func LOAD(t) if(t>=8m,if(t<12m,0.7,0),0)
```

This keeps the schematic visually clean because each behavioral source displays a short expression:

```spice
V=RA(time)
V=RB(time)
V=RC(time)
I=LOAD(time)
```

### 9.4 Simulation Command

```spice
.tran 20m startup
```

This runs a transient simulation from 0 ms to 20 ms and starts from supply startup conditions.

### 9.5 Signals To Plot

```text
V(RAIL_A_1V2)
V(RAIL_B_1V8)
V(RAIL_C_3V3)
V(EN_A)
V(EN_B)
V(EN_C)
I(IC_LOADSTEP)
```

### 9.6 Expected Results

Expected startup:

| Signal | Expected Behavior |
| --- | --- |
| `RAIL_A_1V2` | rises first to 1.2 V |
| `RAIL_B_1V8` | rises after about 2 ms |
| `RAIL_C_3V3` | rises after about 4 ms |
| `EN_A` | turns on at 0 ms |
| `EN_B` | turns on at 2 ms |
| `EN_C` | turns on at 4 ms |
| `I(IC_LOADSTEP)` | active around 8 ms to 12 ms |

### 9.7 Measurement Directives

The simulation includes:

```spice
.meas tran RailA_90pct_time WHEN V(RAIL_A_1V2)=1.08 RISE=1
.meas tran RailB_90pct_time WHEN V(RAIL_B_1V8)=1.62 RISE=1
.meas tran RailC_90pct_time WHEN V(RAIL_C_3V3)=2.97 RISE=1
.meas tran RailC_pre_step_avg AVG V(RAIL_C_3V3) FROM=7m TO=7.8m
.meas tran RailC_load_step_min MIN V(RAIL_C_3V3) FROM=8m TO=9m
.meas tran RailC_recovered_avg AVG V(RAIL_C_3V3) FROM=9m TO=12m
```

These measure:

- when each rail reaches 90% of nominal,
- Rail C voltage before the transient,
- Rail C minimum voltage during the load step,
- Rail C recovered average voltage after the step.

### 9.8 Acceptance Criteria

For this project:

- Rail A should reach 90% before Rail B begins.
- Rail B should reach 90% before Rail C begins.
- Rail C should show controlled droop during the load step.
- The simulation should be readable and repeatable.

For a production design, acceptance criteria would also include:

- ripple voltage limit,
- transient recovery time,
- overshoot limit,
- power-up monotonicity,
- thermal limit,
- efficiency target,
- EMI target.

## 10. KiCad Schematic Design

### 10.1 Main Files

```text
kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_pro
kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_sch
```

### 10.2 Exported Files

```text
kicad/PowerRailsBoard_KiCad/PowerRailsBoard_schematic.pdf
kicad/PowerRailsBoard_KiCad/PowerRailsBoard.svg.png
kicad/PowerRailsBoard_KiCad/PowerRailsBoard.net
kicad/PowerRailsBoard_KiCad/PowerRailsBoard_BOM_from_KiCad.csv
kicad/PowerRailsBoard_KiCad/ERC_report.txt
```

### 10.3 ERC Result

KiCad ERC result:

```text
0 errors, 0 warnings
```

This means the schematic has no KiCad-detected electrical rule problems in the current schematic-level representation.

### 10.4 What The KiCad Schematic Contains

The KiCad schematic includes:

- input connector,
- input reverse-polarity diode,
- input capacitor,
- regulator blocks,
- output capacitors,
- load resistors,
- Rail C P-MOSFET load switch,
- feedback divider representation,
- enable sequencing RC networks,
- test points,
- net labels,
- title notes.

### 10.5 Why Custom Symbols Were Used

The schematic uses custom local symbols in `RailProject.kicad_sym`. This reduces dependency issues. The schematic opens cleanly because it does not rely on missing external symbol libraries.

For a production schematic, the next step would be to replace simplified local regulator blocks with official manufacturer symbols and confirmed footprints.

## 11. BOM Summary

The BOM contains the main components:

| Ref | Part | Purpose |
| --- | --- | --- |
| U1 | TPS62203 | 3.3 V buck reference |
| U2 | TPS62240 | 1.8 V buck reference |
| U3 | TLV70012 | 1.2 V LDO reference |
| Q1 | AO3401 | P-MOSFET Rail C load switch |
| D1 | SS34 | reverse-polarity protection |
| C_IN | 10 uF X7R | input capacitor |
| C_OUT_A/B/C | 22 uF X7R | output capacitors |
| R_LOAD_A | 2.4 ohm | Rail A load |
| R_LOAD_B | 3.0 ohm | Rail B load |
| R_LOAD_C | 4.125 ohm/pulsed | Rail C load representation |

## 12. Important Engineering Caveats

### 12.1 Regulator Current Rating

The most important caveat is that the named parts must be checked against current requirements before fabrication. Model availability does not equal design suitability.

A stronger production design would select regulators with:

- rated output current above the maximum load,
- thermal margin,
- acceptable efficiency,
- available package and footprint,
- datasheet-supported inductor/capacitor values,
- stable control loop over expected load.

### 12.2 LDO Thermal Dissipation

If a 1.2 V rail is generated from 5 V using an LDO at 500 mA:

```text
P = (Vin - Vout) * I
P = (5 V - 1.2 V) * 0.5 A
P = 1.9 W
```

That is too high for many small packages. A real board would likely use a buck converter for 1.2 V or a two-stage approach.

### 12.3 Behavioral Simulation Limits

The behavioral LTspice model does not show:

- switching ripple,
- inductor current ripple,
- compensation loop behavior,
- switch-node ringing,
- layout parasitics,
- EMI,
- real startup soft-start details,
- regulator current limiting.

It is still valuable because it validates architecture and timing.

### 12.4 RC Sequencing Tolerance

RC delays are approximate. Real delays vary due to:

- resistor tolerance,
- capacitor tolerance,
- capacitor leakage,
- temperature,
- enable threshold variation,
- input ramp rate.

For production, a sequencer IC is more reliable.

## 13. PCB Layout Considerations

If this schematic is moved to PCB layout, key rules are:

1. Place input capacitors close to regulator VIN and GND pins.
2. Keep high-current loops short.
3. Keep buck switch nodes small.
4. Route feedback traces away from switch nodes.
5. Use a solid ground plane.
6. Use wide traces or pours for high-current rails.
7. Place output capacitors close to regulator outputs.
8. Keep analog/sensitive nodes away from noisy switching paths.
9. Add thermal copper for regulators and diode.
10. Keep test points accessible.

For a two-layer board, layout is possible but harder. For a cleaner power design, a four-layer board with solid ground and power planes is preferable.

## 14. Bring-Up Plan

### 14.1 Before Power-On

Check:

- visual inspection,
- diode orientation,
- capacitor polarity if any polarized parts exist,
- regulator pinout,
- shorts between each rail and ground,
- continuity of input and ground,
- correct component values.

### 14.2 First Power-On

Use:

- current-limited bench supply,
- low current limit at first,
- oscilloscope on rail test points,
- multimeter for steady-state checks.

Procedure:

1. Apply 5 V with low current limit.
2. Confirm no excessive current.
3. Measure `VIN_5V`.
4. Probe `EN_A`, `EN_B`, `EN_C`.
5. Probe `RAIL_A_1V2`, `RAIL_B_1V8`, `RAIL_C_3V3`.
6. Confirm order and final voltages.
7. Increase load slowly.
8. Test Rail C load transient.

### 14.3 Measurements To Capture

Capture:

- startup sequence waveform,
- rise time of each rail,
- final DC voltage,
- load transient droop,
- recovery time,
- ripple,
- regulator temperature.

## 15. Failure Modes And Debug Strategy

| Symptom | Possible Cause | Debug Step |
| --- | --- | --- |
| Rail missing | regulator disabled, wrong footprint, short, no input | check VIN and EN pin |
| Rail low | overload, wrong feedback, current limit | remove load, inspect feedback |
| Rail noisy | poor layout, missing capacitor, unstable loop | check capacitor placement and values |
| Startup wrong order | RC timing error, threshold variation | probe enable pins |
| Diode hot | excessive current or high voltage drop | measure current and diode drop |
| Rail C droops too much | insufficient capacitance or slow regulator | increase capacitance or choose faster regulator |
| Board current too high | short, wrong part orientation, load too heavy | current-limit and isolate rails |

## 16. Repository Deliverables

| Path | Purpose |
| --- | --- |
| `README.md` | GitHub landing page |
| `ltspice/PowerRails_Behavioral_Complete.asc` | main LTspice visual simulation |
| `ltspice/PowerRails_Behavioral_Complete.cir` | companion netlist |
| `ltspice/PowerRails_REAL.asc` | real-model starter schematic |
| `ltspice/*.asy` | LTspice symbols |
| `ltspice/*.lib` | LTspice model libraries |
| `kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_pro` | KiCad project |
| `kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_sch` | KiCad schematic |
| `kicad/PowerRailsBoard_KiCad/ERC_report.txt` | ERC validation |
| `kicad/PowerRailsBoard_KiCad/PowerRailsBoard_schematic.pdf` | schematic PDF export |
| `kicad/PowerRailsBoard_KiCad/PowerRailsBoard_BOM_from_KiCad.csv` | KiCad BOM export |
| `bom/PowerRails_BOM.csv` | project-level BOM |
| `results/PowerRails_Sequencing_Plot.png` | sequencing simulation plot |
| `results/RailC_Load_Transient_Plot.png` | load transient plot |
| `docs/Exhaustive_Project_Report.md` | detailed project report |
| `docs/Comprehensive_Interview_QA.md` | interview preparation bank |

## 17. What This Project Demonstrates

This project demonstrates:

- power rail planning,
- rail sequencing,
- load calculation,
- transient simulation,
- behavioral modeling,
- schematic organization,
- BOM creation,
- ERC checking,
- test point planning,
- awareness of real-world limitations,
- ability to communicate an engineering design.

The strongest interview point is that the project does not blindly claim perfection. It clearly separates:

- what is proven by the behavioral simulation,
- what is represented by the KiCad schematic,
- what must still be checked before fabrication.

That is how a competent engineer presents a design.

## 18. Short Interview Summary

This project is a multi-rail power management block for an SoC and sensor cluster. It takes a 5 V input and creates 1.2 V, 1.8 V, and 3.3 V rails. The rails are sequenced so the core rail turns on first, then I/O, then sensors. I simulated the startup sequence and a 3.3 V load transient in LTspice, captured an ERC-clean schematic in KiCad, included input protection, enable delay networks, output capacitors, loads, and test points, and documented the design limitations. The next production step would be replacing the learning-model regulator choices with current-rated regulators, assigning final footprints, doing PCB layout, and validating sequencing, ripple, transient response, and temperature on the bench.

## 19. Final Conclusion

The project successfully shows a complete power-management design workflow. It is not just a schematic; it includes requirements, architecture, simulation, schematic capture, validation exports, documentation, and interview preparation.

The design is appropriate as a university-level hardware portfolio project. It shows practical understanding of power sequencing, regulator selection tradeoffs, transient behavior, and bring-up planning. The final design should not be fabricated without regulator current-rating review and thermal analysis, but as an interview project it is technically defensible, well-scoped, and explainable.
