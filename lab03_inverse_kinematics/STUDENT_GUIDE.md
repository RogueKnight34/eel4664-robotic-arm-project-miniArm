# Lab 03 — DH modeling, 3D inverse kinematics and miniArm control

**Start here.** Work in your assigned team with one miniArm. Develop the Craig modified-DH table, run the provided MATLAB simulation, calibrate your arm, and demonstrate an XYZ move. The FK, Jacobian and IK implementations are provided; you do not need to write a new solver.

## 1. Get the files and identify the frames

Download/extract the **complete repository**. Keep `matlab`, `python` and `miniarm_controlC` together in this lab folder. Keep your own measured `calibration.json` when updating files.

![Assigned Craig modified-DH frames on the miniArm](matlab/images/miniarm_craig_dh_frames.png)

Viewed from behind at the Arduino/knobs: **X right, Y forward, Z up**. Positioning joints are **base, shoulder, elbow, wrist**; claw opening is separate. The target is XYZ only; claw orientation is not constrained.

Use the assigned frames to derive each row `[a_(i-1), alpha_(i-1), d_i, thetaOffset_i]`. Explain each length, twist, displacement and zero offset, then specify the fixed wrist-to-tool transform separately. Measure axis-to-axis dimensions and identify the tool center point (TCP). Compare with `matlab/miniarm_model.m`; its fourth column is a fixed offset, added to the changing joint angle.

The default dimensions are H=120, L1=56.5, L2=61.5, L3=85 mm. Use the same measured geometry in MATLAB and the Python GUI's `GEOMETRY`.

**Calculation flow:** DH → link transformations → FK → position error and Jacobian → numerical IK joint updates → calibrated servo commands. See the [MATLAB file/code guide](matlab/README.md) for frame placement, each `.m` file and the loop inside `MiniArmIK.solve`.

## 2. Run MATLAB

Open `lab03_inverse_kinematics/matlab` as MATLAB's current folder, locally or in MATLAB Online. Base MATLAB is sufficient; Robotics System Toolbox is optional.

```matlab
verify_miniarm
verify_collision
demo_ik
demo_ik([-0.040; 0.100; 0.150])
```

The last target is X=-40, Y=100, Z=150 mm. **MATLAB takes metres; the Python GUI takes millimetres.** Inspect the final arm, calculated angles, convergence status and position-error curve. Try another clear target. Different valid joint configurations can reach the same XYZ.

The animation replays numerical solver updates; it is not a timed physical trajectory. Compare with the [expected MATLAB output](matlab/EXPECTED_OUTPUT.md).

## 3. Preview offline in Python

Open a terminal in `lab03_inverse_kinematics/miniarm_controlC`:

```console
python -m pip install -r requirements.txt
python miniarm_controlC.py --simulate
```

The banner must say **SIMULATION — NO PHYSICAL MOVEMENT**. Simulation initializes automatically in IK CONTROL using synthetic calibration; no startup click is required. Synthetic data is not your physical calibration.

1. Enter XYZ in mm, for example `-40, 100, 150`.
2. Click **Calculate / Preview**. Check the result and error curve.
3. Click **Move Arm** after a successful checked preview.

**Solid purple = current estimated pose; dashed blue = target pose; red × = requested XYZ.** The dashed IK target stays fixed while the solid arm moves toward it. Editing XYZ requires a new Calculate / Preview before Move Arm. Camera changes do not move the robot.

APP CONTROL shows Base → Shoulder → Elbow → Wrist → Claw sliders. Slider edits change the dashed target only; Send moves the solid estimate. Returning from IK synchronizes sliders with the current servo-command estimate. Use **Send all five** for the full slider pose; a per-servo Send moves only that servo.

## 4. Connect and calibrate the real arm

**Why calibrate?** IK calculates DH joint angles, while servos accept raw commands. Your measured calibration converts between them. A raw command of 90 does not generally mean a joint angle of 90°. Targets remain XYZ; no polar workspace paper is needed.

Follow the [calibration walkthrough](CALIBRATION_GUIDE.md) for measurement references, five samples per positioning joint, safe raw limits, claw settings, and Check/Save/Reload.

1. Upload `miniarm_controlC/miniarm_controlC.ino` from the GUI folder using the correct board/port, Servo and **FastLED 3.10.3**. Close Serial Monitor.
2. Run `python miniarm_controlC.py --port COM3`, replacing COM3 with your port. Confirm **GUI v2.1.0** and **Firmware 2.1.0**. If connection fails, close other serial applications, check the port and use **Retry connection**.
3. Clear the workspace and support the unloaded arm without entering pinch points. Click **Enable servos one by one**. Shoulder → elbow → wrist → base → claw enable with one-second pauses. Each servo may still move quickly on first enable; unenabled joints can sag. Wait until controls unlock. The claw uses its calibrated startup opening.
4. For a first calibration, use the instructor-supervised **Enable raw neutral (uncalibrated)** action instead. It uses the same sequential enable, but raw 90 has unknown physical joint angles.
5. Run `python miniarm_calibrate.py` alongside the controller. In APP CONTROL, change and **send one joint**, let it settle, and measure its physical DH angle independently. Save your measured `calibration.json`, click **Reload calibration**, and enable again. New files use `"angle_convention": "joint"`.

Once initialized, **Slowly straighten upright** makes a checked, gradual return lasting at least five seconds, longer for large changes. It preserves the claw command. No recalibration is needed merely because you updated from the previous joint-mode calibration software.

## 5. Demonstrate a physical XYZ move

1. Select **IK CONTROL**. Enter a nearby, clear target in mm; start with Z ≥ 60 mm. Use the same frame {0} origin as your simulation.
2. Click **Calculate / Preview**. Check the dashed target, success message, error and proposed move duration.
3. Click **Move Arm** and observe the physical arm and solid estimate moving toward the fixed dashed target. Python sends one final calibrated servo target plus a duration; Arduino generates the intermediate motion.
4. Try another nearby target. Use a longer **Move time (s, minimum)** for slower motion. If a target or path is rejected, choose a clear intermediate/nearby target; do not bypass the checks.

APP/KNOB commands bypass IK collision checking. Mode colors identify APP (green), KNOB (orange) and IK (blue). Avoid touching knobs during IK.

**STOP cancels remaining commands/enables and holds already-enabled servos; it does not remove power or guarantee an instantaneous physical stop.** The display uses commanded-position estimates, not measured joint feedback. Small numerical IK error does not guarantee the same physical accuracy.

If Move Arm is gray, check initialization, measured calibration, firmware version and a fresh successful preview. See the [GUI reference and troubleshooting](miniarm_controlC/README.md).

## 6. Submit these four deliverables

| Deliverable | What to include |
|---|---|
| **DH table derivation** (PDF or Word) | Team names; labeled assigned frames; measured dimensions and units; explanation of how each Craig modified-DH parameter follows from the frames; final table and fixed tool transform. Explain any differences from the default model. A copied table without derivation is insufficient. |
| **MATLAB simulation video** | Show the XYZ input/command, simulation running, final configuration, calculated joint angles and convergence/error result for at least one successful target. |
| **Physical-arm video** | Show the entered XYZ and Calculate / Preview, the **Move Arm** click, and the physical arm moving to the requested position. Keep the GUI and whole arm visible in one recording, or use a synchronized screen/camera view. Identify the frame origin/target location so viewers can assess the physical result. |
| **Measured calibration file** | Submit your arm's actual `calibration.json`, not the synthetic simulation mapping. |

These are the required submissions. No separate full report, report-template completion, screenshot set, experiment-log submission, hand-derived Jacobian, hand-executed IK iteration or pick-and-place video is required. Detailed code and mathematical references are available through the [documentation index](README.md).
