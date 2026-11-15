# Startup and smooth motion — v2.1.0, Ngo Lab

The [student guide](STUDENT_GUIDE.md) is the lab handout; this page describes how the controller communicates with Arduino. Frame assignments, DH, calibration conventions and saved measurements are unchanged.

## Update both sides

Use the complete current lab folder and keep your own `calibration.json`. Upload `miniarm_controlC/miniarm_controlC/miniarm_controlC.ino` with Servo/FastLED 3.10.3, close Serial Monitor, then run the Python GUI. Expect **GUI v2.2.1 / Firmware 2.1.0**. Simulation opens ready in IK mode; its emulator still implements the 2.0.0 timed-motion protocol.

The first hardware enable requires SEQINIT capability. It is not implemented by sending old INIT followed by a sequential move, because INIT would already enable every servo.

## First enable: separate from a known-pose move

The board announces:

```text
FW 2.1.0 TIMED SEQINIT
READY
```

After CFG and MODE A, Python sends:

```text
SINIT gap_ms S1 S2 S3 S4 S5
```

S1..S5 are calibrated raw targets (claw, wrist, elbow, shoulder, base). The integer gap is 1000..5000 ms; the GUI's `STARTUP_SERVO_PAUSE_MS` defaults to 1000. The positioning reference is q=[90,90,0,0] degrees; the first claw target is the calibrated opening. Before calibration, raw 90 is a supervised neutral reference with unknown physical angles.

Arduino enables shoulder, elbow, wrist, base and claw in that order, using a nonblocking timer. It reports INITSEQ progress and returns OK only after the final settling interval. Normal controls remain locked until completion. STOP cancels remaining enables; channels already enabled hold, unenabled channels stay off.

**This spaces out first enable; it does not limit an individual servo's initial physical speed.** No measured starting angles are available. Support the unloaded arm and keep the workspace/pinch points clear. A one-second interval is not feedback that a joint has physically settled.

## Later moves: one final target and a duration

After initialization, XYZ execution and later upright returns use:

```text
T duration_ms S1 S2 S3 S4 S5
```

Python calculates IK, converts the final q through the measured mapping, rounds the positioning commands, preserves claw opening, and validates the coordinated path. It sends only this endpoint and duration. The IK error curve records numerical search updates, not the time trajectory.

Arduino stores current issued commands as the start and uses a common progress variable:

```text
u = clip(elapsed / duration, 0, 1)
s = 3*u*u - 2*u*u*u
raw_i = start_i + s*(final_i - start_i)
```

Channels move together with zero command velocity at the endpoints. Updates are approximately every 20 ms with sub-degree Servo.writeMicroseconds pulses. This is a path in raw-command space, not a straight XYZ line. P replies contain fractional command estimates, not shaft measurements.

| Action | Timing |
|---|---|
| Normal XYZ/manual timed move | Requested minimum (default 2 s), extended to keep cubic peak command speed ≤45 raw units/s |
| Later upright return | At least 5 s, extended to keep cubic peak command speed ≤15 raw units/s |
| First enable | One channel enabled per interval; no claim of speed-controlled motion from unknown angles |

For cubic easing, peak raw speed is `1.5 * max(abs(final-start)) / duration_seconds`. A 90-unit upright change therefore needs at least 9 seconds at the 15-unit/s limit. Firmware accepts integer durations 500..25000 ms and independently rejects invalid, out-of-range or too-fast T requests. These are command-speed limits, not a guarantee of actual mechanical speed or accuracy.

## Path checks and STOP

Python checks raw/joint limits, rounded endpoint miss and modeled capsule/table clearance along the executed common-progress path. Splitting at every channel's calibration knots lets continuous joint-edge checks cover each piece. Cubic easing changes timing, not that geometric path. A blocked move is rejected; choose an intermediate XYZ instead of disabling checks.

The collision approximation does not fully model the camera, claw fingers, loads or surrounding equipment. APP/KNOB moves bypass IK collision checks.

STOP freezes issued commands and cancels active startup/timed/legacy sequences without detaching enabled servos. Python waits for STOPPED and a P reply before unlocking controls. It is not power removal, measured stillness or a guaranteed instantaneous emergency stop. A disconnected controller never automatically replays a motion after reconnecting.

## Validation scope

The instructor reported physical IK working and v2.1.0 sequential startup noticeably gentler on their miniArm. This is an instructor hardware observation, not a guarantee for every arm/calibration.

Earlier automated/host test counts describe earlier revisions; see [maintenance records](docs/maintenance/GUI_VALIDATION.md). This documentation update does not claim a new Arduino build, complete automated regression run or independent physical trial. The [host harness](miniarm_controlC/qa/README.md) complements, but does not replace, checks on the actual board.
