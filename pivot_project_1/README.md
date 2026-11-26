# Pivot Arm Sorter – S7-300 PLC Control with Factory I/O


A beginner automation project: a Siemens S7-300 PLC program that runs a conveyor line in **Factory I/O** and controls a **pivot arm sorter**. When a box reaches the pivot arm, a sensor detects it and the arm turns for 2 seconds to divert the box, then returns to its home position.

\---

## Tools \& Versions

|Tool|Version|
|-|-|
|SIMATIC Manager (STEP 7)|v5.x <!-- TODO: exact version -->|
|PLC|SIMATIC 300 Station – CPU 312|
|S7-PLCSIM|v5.x <!-- TODO: exact version -->|
|Factory I/O|v2.x <!-- TODO: exact version --> – driver: Siemens S7-PLCSIM|
|Programming language|LAD (exported as STL source)|

\---

## Sequence of Operation

1. **Start** – Pressing the Start button latches the `System\_Running` memory bit.
2. **Conveyors run** – While the system is running, conveyors 2, 4 and 6 and the pivot arm belt run continuously.
3. **Box detected** – When `Sensor\_2` detects a box at the pivot arm, the arm turns (`Pivot\_Arm\_Turn`) and holds itself on.
4. **Timed return** – An on-delay timer (T1, 2 s) starts when the arm turns. When it finishes, `Pivot\_Timer\_Done` releases the arm and it returns home.
5. **Stop** – Pressing Stop (normally closed button) breaks the latch and stops the conveyors.

\---

## I/O List

### Inputs

|Address|Symbol|Description|
|-|-|-|
|I10.0|`Start\_PB`|Start push button (NO)|
|I10.1|`Stop\_PB`|Stop push button (NC – signal is 1 when not pressed)|
|I10.3|`Sensor\_2`|Box detection sensor at the pivot arm|
|I10.4|`Sensor\_3`|Sensor 3 – wired, not used in v1.0|

### Outputs

|Address|Symbol|Description|
|-|-|-|
|Q10.0|`Conveyor\_6`|Conveyor 6 motor|
|Q10.1|`Conveyor\_4`|Conveyor 4 motor|
|Q10.2|`Conveyor\_2`|Conveyor 2 motor|
|Q10.3|`Pivot\_Arm\_Turn`|Pivot arm sorter – turn|
|Q10.4|`Pivot\_Arm\_Belt\_Fwd`|Pivot arm sorter – belt forward (+)|

### Internal

|Address|Symbol|Description|
|-|-|-|
|M0.0|`System\_Running`|Start/stop latch – system is running|
|M0.4|`Pivot\_Timer\_Done`|Pivot arm hold time elapsed|
|T1|`Pivot\_Timer`|On-delay timer, 2 s – pivot arm hold time|

\---

## Program Structure

All logic is in **OB1** (main cycle), in 4 networks:

|Network|Function|
|-|-|
|1|Start/stop latch – `(Start\_PB OR System\_Running) AND Stop\_PB = System\_Running`|
|2|Run conveyors 2, 4, 6 and the pivot arm belt while the system is running|
|3|Pivot arm turn – triggered by `Sensor\_2`, self-holding until `Pivot\_Timer\_Done`|
|4|2-second on-delay timer (T1) for the pivot arm hold time|

The full program is available as readable text in [`source/OB1\_source.awl`](source/OB1_source.awl).

\---

## Repository Structure

```
pivot\_project\_1/
├── README.md
├── siemens\_software/
│   └── Pivot\_FP\_v1.0.zip      ← full SIMATIC Manager project archive
├── source/
│   ├── OB1\_source.awl         ← OB1 program as STL text
│   ├── Pivot\_FP\_Symbols.sdf   ← symbol table (I/O list)
│   └── Pivot\_FP\_HW.cfg        ← hardware configuration
├── factory\_io/
│   └── <scene>.factoryio      ← Factory I/O scene
└── media/
    └── screenshots, demo video
```

\---

## How to Run

1. Download `siemens\_software/Pivot\_FP\_v1.0.zip`.
2. In SIMATIC Manager: **File → Retrieve**, select the zip and choose an empty destination folder.
3. Open the project, start **S7-PLCSIM**, set the CPU to **RUN-P**, and download the station (hardware + blocks).
4. Open the scene from `factory\_io/` in Factory I/O.
5. In Factory I/O: **File → Drivers**, select **Siemens S7-PLCSIM**, connect, and check the I/O mapping matches the I/O list above.
6. Start the simulation and press **Start**.

\---

## Known Limitations \& Next Improvements

* **Stop during a pivot move:** if Stop is pressed while the arm is turning, the timer resets but the arm's self-holding branch is not interlocked with `System\_Running`, so the arm can stay turned. Fix planned: put `System\_Running` in series with the whole of Network 3.
* `Sensor\_3` is wired but not used yet.
* Next versions: move the logic into FCs (e.g. FC1 conveyors, FC2 pivot arm), sort boxes by type, and add a WinCC HMI screen.

\---

## Skills Demonstrated

* Start/stop latching (self-holding) circuit
* On-delay timer (S5 timer `SD`)
* Sensor-triggered actuator control
* Testing with S7-PLCSIM and a Factory I/O digital twin
* Project archiving and source export for version control

\---

## Author

**Eng.Saif** – Industrial Automation



