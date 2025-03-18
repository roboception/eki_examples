# Robot Pose Correction Using External Vision

## Overview

This document describes the process for correcting robot positioning errors using external vision measurements. The system uses a camera to detect discrepancies between where the robot believes it is and where it actually is, then calculates a correction transformation that can be applied to future movements.

## Process Description

### Problem

- When the robot is commanded to move to a position P_cmd, it actually moves to a slightly different position P_actual due to calibration errors, mechanical tolerances, etc.
- This discrepancy causes inaccuracies in precise operations like grasping or assembly.

### Solution

1. **Estimate Robot Pose**: Using external vision (cameras and AprilTags), measure where the robot actually is (P_actual) when it thinks it's at a commanded position (P_cmd).
   
2. **Calculate Correction Transform**: Determine the transformation T_correction that maps from desired target positions to the commands needed to reach those targets.
   
3. **Apply Correction**: Use this transformation to adjust future movement commands to compensate for the robot's inherent errors.

### Mathematical Representation

Let:
- P_cmd = Position commanded to the robot
- P_actual = Position where the robot actually ends up (measured by vision)
- T_correction = The correction transformation

During calibration, we measure both P_cmd and P_actual, and determine T_correction such that:
P_cmd = T_correction(P_actual)

For future operations, to reach any target position P_target:
P_cmd = T_correction(P_target)

The beauty of this approach is that we directly measure T_correction during the calibration procedure, eliminating the need for matrix inversions.

## Usage

1. Run the `EstimateRobotPose` program
2. The robot will move to calibration positions
3. Vision system will measure actual positions
4. The correction transformation is calculated
5. Apply this transformation when planning robot movements for improved accuracy 