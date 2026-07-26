# Multi-Rail Power Management Board: Portfolio Summary

## Problem

A small SoC plus sensor cluster needs three clean, correctly sequenced rails from a single 5 V input: 1.2 V for the core, 1.8 V for I/O logic, and 3.3 V for sensors/camera. Incorrect sequencing can power I/O before the core, creating latch-up, back-powering, or undefined boot-state risk.

## Approach

The design uses a 5 V input and creates:

| Rail | Voltage | Load | Purpose | Enable Time |
| --- | ---: | ---: | --- | ---: |
| Rail A | 1.2 V | 500 mA | SoC core | 0 ms |
| Rail B | 1.8 V | 600 mA | I/O logic, IMU | 2 ms |
| Rail C | 3.3 V | 800 mA | Sensors/camera | 4 ms |

The rail order is core first, I/O second, sensors third. Power-down should happen in reverse order.

## Simulation

The LTspice behavioral simulation proves startup sequencing and a Rail C load transient. The main files are:

```text
PowerRails_Behavioral_Complete.asc
PowerRails_Behavioral_Complete.cir
```

Expected plots:

- `PowerRails_Sequencing_Plot.png`: 1.2 V rises first, 1.8 V starts at 2 ms, 3.3 V starts at 4 ms.
- `RailC_Load_Transient_Plot.png`: 3.3 V rail responds to a 0.1 A to 0.8 A load step at 8 ms and stays inside the +/-3% target band.

## Schematic and Bring-Up Planning

The KiCad schematic is included in `PowerRailsBoard_KiCad/` and passes KiCad CLI ERC with 0 errors and 0 warnings. It includes reverse-polarity protection, output capacitors, RC enable-delay networks, a P-MOSFET load switch on the 3.3 V sensor rail, and test points on all rails and enable nets.

## Engineering Caveat

The guide-selected regulator model parts are useful for simulation practice, but current ratings must be checked before fabrication. For a real PCB, I would replace any under-rated regulators with parts rated above the required load current and re-run thermal/transient validation.

## Next Steps

Assign exact footprints, do a rough placement/routing pass, and validate the fabricated board with a scope and electronic load.
