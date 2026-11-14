# Python GUI and Arduino reference — v2.1.0, Ngo Lab

For the required workflow and submissions, use the [student guide](../STUDENT_GUIDE.md). This page explains the interface and resolves common setup problems.

## Install and launch

Keep this folder beside `../python`; install and run with the same Python interpreter:

```console
python -m pip install -r requirements.txt
python miniarm_controlC.py --simulate
```

Simulation opens automatically in IK CONTROL at q=[90,90,0,0] degrees. It opens no serial port and uses synthetic calibration without overwriting your file. To test your measured mapping offline:

```console
python miniarm_controlC.py --simulate --use-calibration
```

For hardware, upload `miniarm_controlC/miniarm_controlC.ino` using Servo and FastLED 3.10.3, close Serial Monitor, then run:

```console
python miniarm_controlC.py --port COM3
```

Use your actual port. Confirm GUI v2.1.0 and Firmware 2.1.0. The simulation emulator may identify its timed-command protocol as 2.0.0; it does not need hardware sequential enable.

The hardware connection opens in a background thread, pulses DTR to reset the board, preserves fragmented serial lines, and waits for FW/READY. Startup retries up to three times, with a ten-second boot timeout per attempt. The editable port field and **Retry connection** let you retry without restarting the GUI. A live disconnection cancels queued commands and requires a retry and fresh initialization; it never replays motion automatically.

## First enable and later upright returns

Connection alone leaves servos off. With a measured calibration, **Enable servos one by one** requests the upright joint reference q=[90,90,0,0] through your own mapping. With no calibration, **Enable raw neutral (uncalibrated)** requests raw 90 on every channel; the physical joint angles are unknown and this is an instructor-supervised calibration step.

Firmware 2.1.0's SINIT command enables **shoulder → elbow → wrist → base → claw**, with one-second pauses. The claw uses its calibrated startup opening. Support the unloaded arm without entering pinch points: each first enable can still move quickly, and unpowered joints may sag. Controls unlock after the final pause/acknowledgement. STOP cancels remaining enables; already-enabled joints hold and others remain off.

After initialization, **Slowly straighten upright** uses the checked coordinated path, preserves APP/IK mode and the claw command, and uses at least five seconds. Larger changes extend the duration to limit the cubic peak command speed to 15 raw units/s. In KNOB mode, choose APP or IK before requesting upright.

The first-enable popup and the old all-at-once INIT workflow are not used for real hardware by this GUI. A firmware without SEQINIT is rejected rather than silently falling back to simultaneous enable. See [motion/protocol details](../SMOOTH_MOTION.md).

## Modes and preview

| Mode | Inputs and behavior |
|---|---|
| APP — green | Base, Shoulder, Elbow, Wrist, Claw sliders; typed raw values; +/- buttons; per-servo and all-servo Send |
| KNOB — orange | Physical knobs control the arm; displays knob/servo values |
| IK — blue | XYZ in mm; Calculate / Preview; Move Arm; error curve and claw control |

Raw sliders are **servo commands, not DH joint angles**. Calibration performs the conversion. APP and KNOB bypass IK collision checks.

- **Solid purple:** current estimated pose, computed from reported servo commands.
- **Dashed blue:** one target pose. There is no second unsent-slider ghost.
- **Red × in IK:** requested XYZ. The dashed executable endpoint includes calibrated command rounding, so its TCP can differ slightly from the red mark.
- In IK, Calculate / Preview sets the dashed snapshot; serial reports move only the solid arm. Editing XYZ/time invalidates Move Arm without moving the retained drawing. Calculate again before executing.
- In APP, editing sliders, +/- or applying typed values updates the dashed preview without sending. Send freezes the endpoint while the solid estimate follows the command sequence. Per-servo Send shows only that channel's intended change; Send all five uses the whole slider pose.
- Switching back to APP synchronizes sliders with the current commands, discarding old unsent edits. Disabled/hidden sliders also receive programmatic updates; deferred Tk callbacks do not create new targets.
- Changing camera or resizing redraws the same world scene. World arrows are frame {0}: X right, Y forward, Z up; they are not all moving DH frames.

These are commanded-position estimates, not encoder measurements. The display cannot detect sag, stall or a physical collision. Claw opening is not modeled as an extra positioning link.

## Calculate, Move and STOP

Calculate uses the shared 3D solver, measured limits and modeled collision clearance. Move sends **one T packet** containing final calibrated targets and duration. Arduino supplies common cubic progress; solver iteration history is not streamed as a trajectory.

The GUI rechecks the start, target, rounded endpoint and direct coordinated path. A rejected path requires a clear intermediate target. Geometry is approximate and does not fully represent the camera, fingers, loads or surroundings.

STOP cancels queued commands, sends the software stop, then waits for STOPPED and a P report. In KNOB it first exits to APP. Servos already enabled retain holding power; STOP is not power removal or a guaranteed instantaneous physical stop. Recalculate an IK plan after stopping.

## Calibration and important files

Read [CALIBRATION_GUIDE.md](../CALIBRATION_GUIDE.md). New files explicitly store `"angle_convention": "joint"` with measured DH angles. Do not copy synthetic data to the real arm or relabel legacy angles without conversion.

| File | Role |
|---|---|
| `miniarm_controlC.py` | Connection, modes, previews and final-target execution |
| `miniarm_ik.py` | Model/units adapters, calibration conversion and executed-path checks |
| `../python/kinematics.py` | Shared modified-DH transforms, FK, Jacobian and numerical IK |
| `miniarm_calibrate.py` | Measurement-entry form; does not move the arm or open a serial port |
| `miniarm_calibration.py` | Calibration validation and forward/inverse interpolation |
| `calibration.json` | Your measured mapping, raw limits and claw settings |
| `miniarm_controlC/miniarm_controlC.ino` | Servo pulses, sequential enable, local interpolation and serial replies |

JSON names `arm` and `forearm` remain compatible aliases for Shoulder and Elbow. Channels remain S1 claw, S2 wrist, S3 elbow, S4 shoulder, S5 base; GUI order is reversed for readability.

## Troubleshooting

| Symptom | Check |
|---|---|
| Missing serial/numpy | `python -m pip install -r requirements.txt` using the same interpreter |
| Missing kinematics | Restore the sibling `../python` folder |
| Port busy/access denied | Close Serial Monitor and all other controller instances; check Device Manager; Retry connection |
| Port opens but no READY | Check baud 115200, correct miniArm sketch/board, USB cable and power; retry after boot timeout |
| Unknown/older firmware or no SEQINIT | Upload the current nested `.ino`, close Serial Monitor, retry; expect Firmware 2.1.0 |
| Repeated READY/reset messages | Check USB/power stability and board resets; do not start a trial until the connection is stable |
| Move Arm gray | Require initialization, measured calibration, IK mode, timed firmware and a fresh successful checked preview |
| Upright bends backward | Verify actual joint zeros/directions against the file; read the logged Upright calibration conversion |
| Works in synthetic simulation only | Saved limits/mapping can differ; inspect calibration, geometry and start pose |
| Tiny residual but poor physical accuracy | Check actual XYZ origin, measured link lengths, calibration, backlash and load |

For optional software checks, run `python -m unittest discover -s ../python -v` from this folder. GUI tests require desktop Tk and must match the current interface; historical test records are in [maintenance](../docs/maintenance/GUI_VALIDATION.md), not additional student work. [Firmware host checks](qa/README.md) do not replace an Arduino build or physical test.

`patch_preview_stability.py` is a legacy one-time updater for older customized GUIs. **Do not run it on this current controller**; the preview changes are already integrated.
