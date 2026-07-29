# TeachGrasps - CADMatch Grasp Pose Teaching Utility

## Quick Overview

For a detailed conceptual explanation of grasp teaching and its use cases, please refer to the [Roboception Grasp Teaching Tutorial](https://tutorials.roboception.de/rc_cadmatch/grasp_teaching.html).

The `TeachGrasps` utility allows you to teach grasp poses on a CADMatch object using the physical robot and gripper.

This is particularly useful when:
- The physical gripper cannot be accurately modeled.
- The target part is complex and dragging/adjusting a grasp through the web interface is difficult.

> [!WARNING]
> **Do not execute this program as is.** You must first teach the grasp poses on the robot.

### General Workflow:
1. **Ensure the physical part can be detected well and is isolated.** It is highly recommended to verify detection using the "Try Out CADMatch" page of the Web GUI. If isolation cannot be achieved, ensure to restrict the detection to a specific region of interest using `TeachingRoi` in the SRC file.
2. **Teach the poses** `T01` to `T10` using the teach pendant (SmartPAD) or WorkVisual (described below). Any taught (non-zero) poses will be registered on the CADMatch template as `taught_grasp_XXX`.
3. **Move the robot completely out of the scene** (camera's view) without touching or moving the part.
4. **Run the program.** The robot will not move. The target part will be detected multiple times, the pose averaged, and the corresponding grasps saved.

---

## Detailed Instructions

### How to Define the Poses (T01 to T10)

You can define the grasp poses using one of two methods:

#### Method A: KUKA SmartPAD Workaround (Touch Up)
To record physical TCP poses without inserting actual motion commands into the program:

0. **Insert a new motion command** at any line in the SRC file.
1. **Rename the point** to `T01` through `T10`.
2. **Press the triangle-button** to open parameter settings and select the active tool (ensure the correct tool number is active). *Note: The Tool Selector in the SmartPAD top status bar is ignored here.*
3. **Press Touch Up** to save the current TCP coordinates to the DAT file, and acknowledge the overwrite prompt.
4. **Press Cancel** on the inline form to discard the motion command from the SRC file, keeping the coordinates saved in the DAT file.

![Touch Up Steps on SmartPAD](teach-pendant-touchup.png)

#### Method B: Direct DAT Editing & WorkVisual
You can also touch up coordinates for variables `XT01` through `XT10` directly inside [TeachGrasps.dat](TeachGrasps.dat) or edit them via WorkVisual. 

*Note: For this method, make sure the correct tool is active in the Tool Selector in the SmartPAD top status bar:*

![Tool Selector](tool-select.png)

---

### Adjusting Gripper Openings During Teaching

Teaching different grasps might require different gripper openings. 

To adjust the physical gripper opening during teaching:
1. Tap/click on one of the gripper move lines in [TeachGrasps.src](TeachGrasps.src#L71-L76):
   - `GripperEGIMoveToAbsPos(opening_3)`
   - `GripperEGIMoveToAbsPos(opening_2)`
   - `GripperEGIMoveToAbsPos(opening_1)`
2. Manually execute only that specific line to change the current gripper opening.
