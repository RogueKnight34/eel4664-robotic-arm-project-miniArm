# miniArm Pick-and-Place Competition

**Two teams · Five blocks each · 60 seconds · Single elimination**

![miniArm competition setup](assets/competition_setup.jpg)

## Setup

- Each team uses a miniArm, a computer with the [Lab 03 GUI](../lab03_inverse_kinematics/miniarm_controlC/README.md), five **3/8-inch (9.525 mm) cubes**, and a cup.
- Before each match, the instructor places blocks randomly within a tested, reachable area in front of each arm. Both stations use comparable layouts.
- Cup center: **X = 127 mm, Y = 0 mm**; bottom at **Z = 0**. Cup height is approximately 127 mm and opening diameter approximately 102 mm.
- Coordinates are measured from the base rotation axis: **+X right, +Y forward, +Z up**, viewed from behind the arm. The supporting tabletop is Z = 0. Release blocks above the cup rim.
- Complete calibration and practice before the match. The image illustrates the setup; it is not to scale.

## Match rules

1. At **Go**, the 60-second timer starts. Teams may then inspect the new layout, estimate block coordinates **by eye**, and enter them into the GUI.
2. Pick up blocks **one at a time** and place them into the cup. Teams may choose the order and retry missed blocks.
3. The first team to place all five blocks in the cup wins. Otherwise, the team with more blocks in the cup at 60 seconds wins.
4. Only blocks fully inside and remaining in the cup by the deadline count. Do not touch the robot, blocks, or cup during play.
5. For a tie, reset with one block at the same relative location for each team. First to score within 60 seconds wins; repeat if needed.

**Code modifications are allowed:** Students may freely modify or replace the GUI, control code, Arduino firmware, motion timing, grip/settling settings, and planning approach, including removing the planner or its checks. Teams are responsible for testing their changes and operating the robot safely.

Keep hands clear while the arm moves. Stop the match and notify the instructor if intervention is needed.

## Tournament

The instructor draws a single-elimination bracket. If needed, some teams will advance automatically to the second round without playing a first-round match. Each match winner advances; the final winner is champion.
