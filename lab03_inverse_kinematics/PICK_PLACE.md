# Block-to-cup task — GUI v2.2.1

**Move Arm remains a single XYZ move.** The separate Preview Pick & Place / Run Pick & Place buttons automate a checked sequence. Complete this task after the single-target XYZ move and submit one continuous physical video demonstrating **at least three randomly placed blocks** picked up one at a time and placed into the cup, as specified in the [student guide](STUDENT_GUIDE.md#7-submit-these-five-deliverables). Estimate and enter each new pickup target; repeat Preview then Run for each block. There is no time limit for the lab video.

[Instructor demonstration: block pickup and release into a cup](https://youtube.com/shorts/IMDFx-MZY2A?feature=share)

## Why the whole task is checked

Solving the pickup point establishes only one configuration. The lift, transport and release points require their own IK solutions, and transitions must respect limits and cup/robot clearance. A low pickup may be reachable while its higher approach is not. An endpoint can also be clear while the path to it crosses the cup.

The preview button runs the full preflight without sending servo commands. A visible preview and a separate Run are interface choices; the full checks are necessary even if a future interface combines those actions. This lab follows the existing Preview then Run workflow.

## Install

Update `miniarm_controlC.py` and put the new `miniarm_pick_place.py` alongside it in the existing `lab03_inverse_kinematics/miniarm_controlC` folder. Keep the other modules, your measured `calibration.json` and the sibling `python` folder. Existing firmware **2.1.0** already supports the required timed commands; no new Arduino upload or recalibration is needed.

The update ZIP contains these two modules, a copy of this guide and focused regression tests. Do not replace or copy a synthetic calibration onto your physical arm.

## Your cup and coordinate system

All dimensions below are in world frame {0}, measured from the base-axis/table origin: X right, Y forward, Z up.

| Setting | Default |
|---|---:|
| Cup center X | 127 mm (5 inches) |
| Cup center Y | 0 mm |
| Cup base Z | 0 mm |
| Cup rim Z | 127 mm (5-inch height) |
| Opening diameter | 101.6 mm (4 inches) |
| Lift / transport clearance | 40 mm |
| Held-block radius about the TCP | 30 mm — illustrative; measure your block/grasp |
| Extra release margin | 15 mm |
| Settling / grip pause | 0.8 seconds |

**Cup settings...** changes these values. The held-block radius must enclose the entire held object about the tool center point, including its offset from that point. It is not inferred from the photo. The planner requires room for that radius plus positioning margin within the opening.

The nominal release point is `[127, 0, 172]` mm with the defaults: rim height + block radius + release margin. Z=0 is the cup bottom/table, **not** the release target. Transport is above both the pickup and release heights. Large heights/clearances can make a task unreachable; the planner then refuses execution.

## Claw presets

Use **Claw settings...** to edit/test the raw Open and Closed values used by this sequence. Apply is session-only; Save to calibration preserves your measured joints and makes a backup. Test buttons move only the claw and are disabled during a task. Editing presets invalidates an old task preview, so preview the full sequence again before Run.

## Use

1. Start with `python miniarm_controlC.py --simulate` for a dry run. A reproducible software example is pickup `[0,130,50]` mm; this is not a measurement of your real block.
2. In IK CONTROL, enter the block's **TCP grasp point** in the existing XYZ fields. This is where the claw should close, not the block's bottom corner or the table surface.
3. Open **Cup settings...** and check the cup position, rim height, diameter, held-block radius and clearances. Start with an empty claw.
4. Click **Preview Pick & Place**. Planning runs in the background and sends no servo commands. Inspect the cup drawing, stage list, dashed pickup pose and schematic TCP endpoint route.
5. If every stage passes, click **Run Pick & Place**. The sequence is:
   - Open claw.
   - Approach above the block.
   - Lower to the pickup point.
   - Close claw and pause.
   - Lift the block.
   - Transfer above the cup.
   - Lower to the release point above the rim.
   - Open claw and pause.
   - Retreat upward; no automatic return through the cup.
6. For hardware, **click Enable servos one by one after connection** and wait for controls to unlock (instructor-guided raw-neutral enable before the first calibration). Initialize/calibrate as in the student guide and test the route **without the block first**. Then try the actual pickup while watching the physical arm and workspace.

Each stage sends one timed final-target packet when its predecessor is acknowledged; extra checked waypoints are used only if a direct stage cannot be planned. Arduino generates the smooth interpolation. Settling pauses use Tk timers, so STOP stays available. The dashed arm is the active stage's fixed target; the solid arm follows reported commands. The task does not silently change Move Arm into a grasping action.

## Practical points that make the demonstration work

- Measure pickup/cup coordinates from the base-axis/table origin; secure the base, cup and block so their positions stay fixed.
- Calibrate joint signs/zeros and verify TCP height before the task. Enter the claw's grasp point, not the table or a corner of the block.
- Test Open/Closed presets with the real lightweight block. Close enough to hold it without forcing the mechanism; save the values used in the video.
- Check that the selected XYZ solution actually aligns the claw with the block. Free pitch can make a reachable point unsuitable for gripping.
- Use your measured opening diameter, rim height and held-object radius. Leave room above the rim; do not set release Z to the cup's bottom.
- Keep every approach/lift/transfer point reachable. Large clearance can exceed reach; adjust the physical setup and measured settings rather than disabling checks.
- Use a dry run without the block, adequate settling time, stable power/USB and a clear route. Keep hands out of the route while executing.
- After a reset/READY, loss of connection, moved setup or changed presets, initialize as required and preview again. Do not touch the knobs during IK; takeover cancels the task.

## Checks and limits

All stages are solved and checked before Run is enabled. Checks include calibrated/nominal limits, robot/table/self clearance, command rounding, and a conservative solid envelope around the cup. The cup envelope is an axis-aligned box enclosing its opening/body with a 5 mm allowance. The held block is represented by a TCP-centered sphere; these deliberately approximate the geometry. Continuous path checks split at calibration knots and use conservative travel bounds, rather than checking only a few snapshots.

The route uses Cartesian stage endpoints, but interpolation between them occurs in calibrated servo-command space. The thin teal route only connects stage endpoints; it is not the exact continuous TCP path.

**XYZ-only IK does not set claw pitch or verify a grasp.** The fingers, cup wall/contact dynamics and an inaccurately specified held object are not fully modeled. A complete software sequence does not establish that the block was gripped or landed in the cup. Adjust the block placement/grasp point if the claw orientation is unsuitable; do not bypass rejected paths.

Editing inputs invalidates Run. A changed starting pose, STOP, controller reset or disconnection cancels the remaining task and requires a fresh preview. STOP keeps the current claw command rather than automatically opening a possibly loaded gripper; enabled servos keep holding power. It is not a guaranteed instantaneous physical emergency stop.

## Validation

From the GUI folder:

```console
python -m unittest test_pick_place -v
python -m unittest discover -s ../python -v
```

For this revision, **22 planner/native-Tk/emulator tests and 16 core tests passed** on Windows with Python 3.14. They cover the full command sequence, opposite wrist calibration direction, no serial movement during planning, preflight rejection, stale inputs/start, cancellation generations, STOP during the grasp pause, disconnection, one target ghost and the original Move Arm action. Python sources also compiled successfully. These are focused software checks; no physical pick-and-place trial was performed by the assistant.

Claw-preset update v2.2.1: 16 additional data/persistence/native-GUI tests passed. The combined 38 claw/task tests and 16 core tests passed (54 checks); no physical grip-force or pick-and-place verification is claimed.
