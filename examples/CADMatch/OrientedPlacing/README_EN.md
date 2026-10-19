# OrientedPlacing - CADMatch Pick and Place in a Fixed Orientation

`OrientedPlacing` picks objects lying randomly in a bin and places them on a stack, **all in the same object orientation**, no matter which grasp was used or how the object was lying.

> [!WARNING]
> **Do not execute this program as is.** Teach the poses and implement the gripper functions first, and test in T1.

## Requirements

- rc_cube / rc_visard with CADMatch and the **`rc_cadmatch-detect_object_extended`** service (newer firmware). Only this service returns the object (match) pose for every grasp; the plain `detect_object` does not.
- The EKI XML files for `detect_object_extended`, `start` and `stop` on the controller.
- A CADMatch template with grasps defined (e.g. with [TeachGrasps](../TeachGrasps/README_EN.md)). The more grasps the template has, the more often a reachable stack placement is found.

## How it works

1. Detect objects with `detect_object_extended`. For each grasp the program reads the gripper pose and the object pose.
2. Compute the object pose in the gripper (match-in-hand): `INV_POS(grasp) : match`.
3. From the desired object pose on the stack, compute the gripper pose for placing: `stack_pose : INV_POS(match_in_hand)`.
4. Check with `INVERSE()` whether the robot can reach that gripper pose. The axis configuration at the pick is used as reference, so the robot stays in the same configuration and does not flip.
5. The first grasp (up to `MAX_GRASP_CHECKS`) with a reachable stack pose is picked and placed. Every placed object shifts the next stack pose by `F_STACK_OFFSET`.
6. If no grasp allows a reachable stack pose, the object is picked with the first grasp and dropped at `OVER_BIN_POSE` (e.g. a rework bin).
7. Repeat until no more objects are detected or `MAX_STACK_ITEMS` is reached.

## Setup

In the `.src` file:
- Service names, `TEMPLATE_ID`, and optionally `LOAD_CARRIER_ID`, `ROI_ID` and `GRIPPER_ID`.
- `GripperInit()`, `GripperOpen()` and `GripperClose()` for your gripper.

In the `.dat` file:
- `STARTPOSE`: a pose from which the camera sees the bin.
- `OVER_BIN_POSE`: where objects are dropped that cannot be placed in the stack orientation.
- `F_STACK_START_POSE`: the **object** pose of the first stack place, in the robot base (not the gripper pose). The easiest way to get it: place one object by hand where the first object should go, run a detection on it, and use the reported match pose.
- `F_STACK_OFFSET`: shift between two stack places (X, Y, Z in mm).
- `fDatDefault`: your tool and base numbers.

## Notes

- `INVERSE()` only checks the kinematics (axis limits, singularities). It does not check collisions on the way or at the place pose.
- The pre-grasp offset is applied along the gripper Z axis; the pre-place offset is a height above the place pose in the base.
