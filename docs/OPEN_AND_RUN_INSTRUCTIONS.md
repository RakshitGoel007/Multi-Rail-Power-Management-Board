# How To Open and Run the Project on Mac LTspice

Use this GitHub project folder:

```text
Multi-Rail-Power-Management-Board
```

## Best File To Open

Open this file first:

```text
ltspice/PowerRails_Behavioral_Complete.asc
```

This is the safest file because it does not need custom symbols.

## If Double-Click Does Not Work

1. Open LTspice first.
2. Click:

```text
File -> Open
```

3. Go to:

```text
Multi-Rail-Power-Management-Board -> ltspice
```

4. Select:

```text
PowerRails_Behavioral_Complete.asc
```

5. Click Open.

## Run Simulation

Click the running-man button.

## Plot These Signals

In the waveform window, add these traces:

```text
V(rail_a_1v2)
V(rail_b_1v8)
V(rail_c_3v3)
```

Also useful:

```text
V(en_a)
V(en_b)
V(en_c)
I(ic_loadstep)
```

## What You Should See

- 1.2 V turns on first at 0 ms.
- 1.8 V turns on second at 2 ms.
- 3.3 V turns on third at 4 ms.
- Rail C load current jumps at 8 ms.

## Files Not To Start With

Do not start with:

```text
old downloaded files
ltspice/PowerRails_REAL.asc
```

Those are older or real-model starter files. Use the behavioral complete file first.

## KiCad Project

Open this in KiCad:

```text
kicad/PowerRailsBoard_KiCad/PowerRailsBoard.kicad_pro
```

The KiCad schematic has already been checked with KiCad CLI:

```text
0 errors, 0 warnings
```

Exported KiCad files are also inside `kicad/PowerRailsBoard_KiCad/`:

```text
PowerRailsBoard_schematic.pdf
PowerRailsBoard.net
PowerRailsBoard_BOM_from_KiCad.csv
ERC_report.txt
```
