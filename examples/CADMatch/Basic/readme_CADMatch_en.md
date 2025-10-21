# KUKA KRL Example: CADMatchObjectDetection

## 1. Overview

This KUKA KRL program (`CadMatchExample.src`) demonstrates CAD-based object detection and pick-and-place operations using Roboception's rc_visard or rc_cube with the CADMatch software module. The robot identifies objects based on a predefined CAD template and attempts to pick them up and place them at a specified location.

The program follows these main steps:
1.  Initializes the robot and EKI (EtherNet/KRL Interface) communication.
2.  Starts the CADMatch service on the Roboception device.
3.  Enters a loop to continuously detect and process objects:
    1.  Configures detection parameters (e.g., template ID, pose frame, region of interest).
    2.  Triggers the object detection service.
    3.  Includes a retry mechanism if detection fails.
    4.  If an object is successfully detected, it reads the grasp information and the associated `gripper_id`.
    5.  Determines the target gripper opening based on the received `gripper_id`.
    6.  Additionally, it reads the `grasp_id` for the detected object.
    7.  Selects a specific `PLACEPOSE` (e.g., `PLACEPOSE1`, `PLACEPOSE2`) based on the `grasp_id`.
    8.  Executes a pick-and-place sequence using the selected `PLACEPOSE`:
        1.  Moves to a pre-grasp position above the object.
        2.  Moves to the grasp position.
        3.  Closes the gripper (using the determined opening).
        4.  Moves to a pre-place position.
        5.  Moves to the final place position.
        6.  Opens the gripper.
    9.  Returns to the start position and repeats the cycle.
4.  Stops the CADMatch service upon program termination.

## 2. Prerequisites

*   KUKA KRC4 robot controller.
*   Roboception rc_visard or rc_cube equipped with the CADMatch software module.
*   Proper EKI configuration on the KUKA controller and the Roboception device. The service names in the KRL program must match the EKI XML configuration.
*   Network connection established between the KUKA controller and the Roboception device.
*   The specified CAD template (`TEMPLATE_ID`) must be uploaded and configured in the CADMatch module on the Roboception device.
*   A suitable gripper mounted on the robot, with `GripperOpen()` and `GripperClose()` functions correctly implemented in this KRL program.

## 3. Configuration Parameters

The following parameters at the beginning of `CadMatchExample.src` need to be configured before running the program:

*   **EKI Service Names:**
    *   `serviceNameDetect[]`: EKI service name for object detection (e.g., `"rc_cadmatch-detect_object"`).
    *   `serviceNameStart[]`: EKI service name to start the CADMatch module (e.g., `"rc_cadmatch-start"`).
    *   `serviceNameStop[]`: EKI service name to stop the CADMatch module (e.g., `"rc_cadmatch-stop"`).
*   **Detection Parameters:**
    *   `TEMPLATE_ID[]`: Name of the CAD template to be matched (e.g., `"Template_Name"`). This must match the template ID on the rc_visard/rc_cube.
    *   `POSE_FRAME[]`: Reference frame for the reported poses (e.g., `"external"` for poses in the robot's base frame, assuming proper hand-eye calibration).
    *   `LOAD_CARRIER_ID[]`: ID of a pre-configured load carrier on the Roboception device. Use `" "` (whitespace) if no load carrier is used.
    *   `ROI_ID[]`: ID of a pre-configured Region of Interest (ROI) on the Roboception device. Use `" "` if no specific ROI is used.
*   **Collision Checking Parameters (Optional):**
    *   `GRIPPER_ID[]`: ID of a gripper model configured on the Roboception device for collision checking. Use `" "` to disable.
    *   `PRE_GRASP_OFFSET_X`, `PRE_GRASP_OFFSET_Y`, `PRE_GRASP_OFFSET_Z`: Offsets (in meters) from the grasp point used by the rc_visard for collision checking of the pre-grasp approach. Typically, Z is negative. Set to `0.0` to disable individual offsets.
*   **Motion Parameters (in millimeters):**
    *   `MOT_PRE_GRASP_Z_OFFSET`: Vertical offset (e.g., `-300` mm) from the grasp pose for the robot's pre-grasp approach position.
    *   `MOT_PRE_PLACE_Z_OFFSET`: Vertical offset (e.g., `-300` mm) from the place pose for the robot's pre-place approach position.
*   **Detection Retry Configuration:**
    *   `MAX_DETECTION_RETRIES`: Maximum number of times the program will attempt detection if it fails (e.g., `3`).
*   **Gripper Configuration:**
    *   `defaultOpening`: Default opening value for the gripper (e.g., `100.0`). This is used if no specific `gripper_id` is matched from the detection results.

## 4. Taught Positions

The following positions need to be taught in the associated `.dat` file (`CadMatchExample.dat`):

*   `STARTPOSE`: The robot's starting/home position.
*   `PLACEPOSE`: The default target position where the object will be placed if no specific `grasp_id` matches.
*   `PLACEPOSE1`, `PLACEPOSE2`: Alternative target positions, selected based on the `grasp_id` of the detected object.

## 5. Gripper Control

The program uses two functions for gripper control:

*   `GripperClose()`: Implement the logic to close your specific gripper.
*   `GripperOpen(targetOpening : REAL)`: Implement the logic to open your specific gripper to the `targetOpening` value. This value is determined based on the `gripper_id` received from the CADMatch module or defaults to `defaultOpening`.

You must adapt the example lines within these functions (e.g., `SetOutput(...)` or `SetAnalogOutput(...)`) to match your gripper's interface.

## 6. How it Works

1.  **Initialization:** The robot moves to `STARTPOSE`, opens the gripper to `defaultOpening`, and initializes EKI connections for the CADMatch services. The CADMatch application on the sensor is started via `serviceNameStart`.
2.  **Detection Loop:**
    *   The program attempts to detect objects using `PerformDetection`. This function sends the configured parameters (template ID, ROI, collision parameters, current robot flange pose) to the `serviceNameDetect` EKI service.
    *   If detection fails, it retries up to `MAX_DETECTION_RETRIES` times, displaying the error message from the vision system on each attempt.
    *   If detection is successful, `RC_READRESULT` parses the response. The program currently focuses on the first detected grasp (`DETECTEDGRASPS[1]`). It reads the `gripper_id` associated with this grasp.
    *   **Gripper ID Logic:** Based on the `RECEIVED_GRIPPER_ID[]` (e.g., "gripper_a", "gripper_b"), a `targetOpening` value is set. If the ID is unknown or "none", `defaultOpening` is used.
    *   The `grasp_id` (e.g., "grasp_1") is also read into `RECEIVED_GRASP_ID[]` and printed to the KCP.
    *   **Place Pose Selection:** Based on the value of `RECEIVED_GRASP_ID[]`, a `selectedPlacePose` is chosen (e.g., `PLACEPOSE1` if "grasp_1", `PLACEPOSE2` if "grasp_2", otherwise the default `PLACEPOSE`).
3.  **Pick and Place (`ExecutePickAndPlace`):**
    *   **Pre-Grasp:** The robot moves to `PREGRASPPOSE`, which is calculated by applying `MOT_PRE_GRASP_Z_OFFSET` to the `Z` coordinate of the detected `GRASPPOSE`.
    *   **Grasp:** The robot moves linearly to the `GRASPPOSE`.
    *   **Close Gripper:** `GripperClose()` is called.
    *   **Lift:** The robot moves back to `PREGRASPPOSE`.
    *   **Pre-Place:** The robot moves to `PREPLACEPOSE`, calculated by applying `MOT_PRE_PLACE_Z_OFFSET` to the `Z` coordinate of the taught `PLACEPOSE`.
    *   **Place:** The robot moves linearly to `PLACEPOSE`.
    *   **Pre-Place:** The robot moves to `PREPLACEPOSE`, calculated by applying `MOT_PRE_PLACE_Z_OFFSET` to the `Z` coordinate of the `selectedPlacePose`.
    *   **Place:** The robot moves linearly to the `selectedPlacePose`.
    *   **Open Gripper:** `GripperOpen(targetOpening)` is called, using the determined opening value.
    *   **Retreat:** The robot moves back to `PREPLACEPOSE`.
    *   **Return Home:** The robot moves to `STARTPOSE`.
4.  **Loop Continuation:** The program loops back to attempt another detection.
5.  **Termination:** If detection fails after all retries, the program halts and exits the loop. The `RC_STOP` function is called to stop the CADMatch service on the Roboception device before the KRL program ends.

## 7. Customization

*   **Multiple Objects:** To process more than one detected object per cycle, modify the loop in `DEF CadMatchExample()` to iterate from `1` to `NUMDETECTEDGRASPS` and call `ExecutePickAndPlace` for each valid grasp.
*   **Gripper Logic:** Adapt `GripperOpen()` and `GripperClose()` for your specific hardware.
*   **Error Handling:** Enhance error handling as needed for your application.
*   **Motion Parameters:** Adjust velocities, accelerations, and approximation parameters in the PTP and LIN motion commands for smoother and safer operation.
*   **Gripper ID to Opening Mapping:** Modify the `IF STRCOMP(...)` block in the main loop to match the `gripper_id` strings returned by your CADMatch setup and set appropriate `targetOpening` values.
*   **Grasp ID to Place Pose Mapping:** Modify the `IF STRCOMP(...)` block related to `RECEIVED_GRASP_ID[]` to match the grasp IDs returned by your vision system and ensure corresponding `PLACEPOSE1`, `PLACEPOSE2`, etc., are defined in the `.dat` file and taught. 