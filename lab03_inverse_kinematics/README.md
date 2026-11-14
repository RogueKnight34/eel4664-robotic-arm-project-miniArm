# Lab 03 — miniArm 3D inverse kinematics

**Students: follow [STUDENT_GUIDE.md](STUDENT_GUIDE.md).** This is the main handout and the authoritative list of required submissions: DH derivation, MATLAB simulation video, physical XYZ-move video and measured calibration file.

Current controller/firmware: **GUI v2.2.0 / firmware v2.1.0**, Ngo Lab. First hardware enable is sequential; later upright moves and XYZ moves use Arduino interpolation. Keep the complete lab directory structure and your own calibration.

## Read only what you need

| Document | Purpose | Who needs it? |
|---|---|---|
| [Student guide](STUDENT_GUIDE.md) | Ordered lab steps and the four deliverables | All students |
| [Calibration guide](CALIBRATION_GUIDE.md) | Why calibration is necessary; angle references; measurement, save/reload and validation | Students using hardware |
| [MATLAB code guide](matlab/README.md) | Assigned frames, DH, each `.m` file, transformations, FK, Jacobian and IK execution flow | Code/reference reading |
| [Expected MATLAB output](matlab/EXPECTED_OUTPUT.md) | Figures and results to compare with a successful simulation | Simulation reference |
| [Python GUI reference](miniarm_controlC/README.md) | Modes, previews, startup, connection troubleshooting and file roles | Hardware/GUI reference |
| [Optional block-to-cup task](PICK_PLACE.md) | Previewed grasp, lift, transfer, release and retreat | Optional extension; not a submission |
| [Motion and startup details](SMOOTH_MOTION.md) | Sequential enable, final-target packets, interpolation, timing and STOP | Optional technical detail |
| [Jacobian walkthrough](matlab/JACOBIAN_WALKTHROUGH.md) | Product-rule partial derivatives and the two code loops | Optional deeper study |
| [Optional Jacobian verification](OPTIONAL_JACOBIAN_VERIFICATION.md) | Numerical derivative checks and an IK iteration exercise | Optional enrichment; not a submission |
| [MATLAB/Python pipeline audit](PYTHON_MATLAB_AUDIT.md) | Cross-language conventions, calibration equations and implementation mapping | Instructor/advanced reference |
| [Collision model](COLLISIONS.md) | Capsule checks and their limitations | Optional implementation detail |

The reference files keep the main handout short; they are **not additional assignments**. Pick-and-place is an optional extension for this lab, not a required video.

## Maintenance history

Historical [code review](docs/maintenance/CODE_REVIEW.md), [simulation validation](docs/maintenance/VALIDATION.md) and [GUI validation](docs/maintenance/GUI_VALIDATION.md) are grouped under `docs/maintenance/`. They preserve earlier design/test evidence; their old version descriptions are not current operating instructions. Firmware host-test instructions remain in [miniarm_controlC/qa](miniarm_controlC/qa/README.md).

`Lab03_Report_Template.docx` is retained as an optional legacy worksheet. Students do not need to complete it; use the deliverables at the end of STUDENT_GUIDE.md.
