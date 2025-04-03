# Robot Pose Correction Using External Vision

## Overview

This document describes the process for correcting robot positioning errors using external vision measurements. The system uses a Roboception camera (i.e., rc_visard or rc_viscore) to detect discrepancies between where the robot believes it is and where it actually is, then calculates a correction transformation that can be applied to future movements.

## Process Description

### Problem

- When the robot is commanded to move to a position P_cmd, it actually moves to a slightly different position P_actual due to calibration errors, mechanical tolerances, and issues in the hand-eye-calibration.
- This discrepancy causes inaccuracies in precise operations like grasping or assembly.

### Solution

1. **Estimate Robot Pose**: Using external vision (Roboception cameras and AprilTags), measure where the robot actually is (P_actual) when it thinks it's at a commanded position (P_cmd).
   
2. **Calculate Correction Transform**: Determine the homogeneous transformation matrix T_correction that maps from desired target positions to the commands needed to reach those targets.
   
3. **Apply Correction**: Use this transformation matrix to adjust future movement commands to compensate for the robot's inherent errors.

### Mathematical Representation

Let:
- P_cmd = Position commanded to the robot (homogeneous transformation)
- P_actual = Position where the robot actually ends up (measured by vision)
- T_correction = The correction transformation matrix

During calibration, we measure both P_cmd and P_actual, and determine T_correction such that:
P_cmd = T_correction * P_actual

This is accomplished by multiplying both sides by the inverse of P_actual:
T_correction = P_cmd * P_actual^(-1)

For future operations, to reach any target position P_target:
P_cmd = T_correction * P_target

### Implementation Details

The calibration and correction process consists of these key steps:

1. **Calibration Phase**:
   - The robot moves to a known position (PrePointPose) where AprilTags are visible to the Roboception camera
   - The system measures where the robot thinks it is (P_cmd) using `P_cmd = $POS_ACT_MES : INV_POS($TOOL)`
   - External vision determines where the robot actually is (P_actual)
   - The correction transformation is calculated using `T_correction = P_cmd : INV_POS(P_actual)`
   - This calculation gives us the transformation that converts from actual space to command space

2. **Application Phase**:
   - For any desired target position (PointPose), we calculate a corrected command position
   - This is done by applying `CorrectedPointPose = T_correction : PointPose`
   - When the robot moves to CorrectedPointPose, it will actually end up at the desired PointPose
   - The system effectively pre-compensates for the robot's systematic errors

3. **Verification**:
   - The system demonstrates the correction by first moving to the uncorrected position (PointPose)
   - Then it moves to the corrected position (CorrectedPointPose)
   - The user can visually confirm that the corrected position achieves better accuracy

In KRL robot programming language, the `:` operator represents frame multiplication, and the order matters. The correction works because we're using the same systematic error relationship that was measured during calibration to pre-compensate for future movements.

## Usage

1. Run the `EstimateRobotPose` program
2. The robot will move to calibration positions
3. Vision system will measure actual positions
4. The correction transformation matrix is calculated
5. The robot will demonstrate both uncorrected and corrected movements
6. Apply this transformation matrix when planning robot movements for improved accuracy 