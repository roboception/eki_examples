# CADMatch with Load Carrier Detection

## Overview

This example demonstrates CAD-based object detection combined with load carrier detection. It addresses a common challenge: when the robot must be positioned far from the load carrier to capture the entire carrier in the camera view, objects appear very small, reducing detection accuracy.

The solution uses a three-step workflow: First, detect the load carrier from a far position where the entire carrier is visible. Second, save the detected pose with `EXACT_POSE` in the rc_cube database. Third, perform CADMatch detection using this exact pose - the system no longer needs to re-detect the load carrier, allowing the robot to move closer for better object detection accuracy

## Prerequisites

Before running this program, ensure:

1. **Hand-Eye Calibration Complete:**
   - Required for `pose_frame: "external"` to work
   - Use the HandEyeCalibration example or Web UI
   - Without calibration: "External pose frame is not possible" error

2. **CADMatch Template Uploaded:**
   - CAD model must be uploaded to rc_cube
   - Template ID must match configuration in code

## What's Included

### Core Files (Upload to Robot)
1. **CADMatch-Loadcarrier.src** - Main program
2. **CADMatch-Loadcarrier.dat** - Data declarations

## Setup Instructions

### 1. Configure Parameters in .src file
```krl
; Service names - must match EKI XML configuration files
serviceNameDetectLC[] = "rc_load_carrier-detect_load_carriers"
serviceNameSetLC[] = "rc_load_carrier_db-set_load_carrier"
serviceNameDetect[] = "rc_cadmatch-detect_object"

; Configuration parameters
TEMPLATE_ID[] = "YourTemplateID"              ; Your CAD template ID
POSE_FRAME[] = "external"                      ; Reference frame
LC_ID_TEMPLATE[] = "your_lc_model" ; LC to detect in step 1
LC_ID_EXACT[] = "lc_exact"          ; LC ID to save with exact pose
```

### 2. Teach Robot Positions
In the `.dat` file, teach:
- **STARTPOSE** - Position where the entire load carrier is visible

### 3. Create Load Carriers in rc_cube (IMPORTANT)
Before running the program, create **TWO** load carriers in the rc_cube database:

**Via Web UI (Recommended):**
1. Navigate to: Database → Load Carriers
2. Create first load carrier:
   - ID: `lc_generic` (or your choice for `LC_ID_TEMPLATE`)
   - Enter approximate dimensions of your physical carrier
3. Create second load carrier:
   - ID: `lc_exact` (or your choice for `LC_ID_EXACT`)
   - Use the **same dimensions** as the first carrier
   - This one will be automatically updated by the program with the exact detected pose

**Via REST API:**
```bash
# Create template load carrier for detection
curl -X PUT http://YOUR_RC_CUBE_IP/api/v2/nodes/rc_load_carrier_db/services/set_load_carrier \
  -H 'Content-Type: application/json' -d '{
  "args": {
    "load_carrier": {
      "id": "lc_generic",
      "inner_dimensions": {"x": 0.276, "y": 0.176, "z": 0.12},
      "outer_dimensions": {"x": 0.3, "y": 0.203, "z": 0.125}
    }
  }
}'

# Create exact load carrier (will be updated by program)
curl -X PUT http://YOUR_RC_CUBE_IP/api/v2/nodes/rc_load_carrier_db/services/set_load_carrier \
  -H 'Content-Type: application/json' -d '{
  "args": {
    "load_carrier": {
      "id": "lc_exact",
      "inner_dimensions": {"x": 0.276, "y": 0.176, "z": 0.12},
      "outer_dimensions": {"x": 0.3, "y": 0.203, "z": 0.125}
    }
  }
}'
```

**Why two load carriers?**
- `lc_generic`: Used for initial detection (Step 1) - needs approximate dimensions
- `lc_exact`: Updated by program with exact detected pose (Step 2) - used for all subsequent CADMatch detections

### 4. Upload Files to Robot
Upload program files to the robot:
- `CADMatch-Loadcarrier.src`
- `CADMatch-Loadcarrier.dat`

## How It Works

```
Step 1: Detect Load Carrier
   └─► Returns: pose, dimensions

Step 2: Save with EXACT_POSE
   └─► Stores in database with pose_type="EXACT_POSE"

Step 3: CADMatch Detection
   └─► Uses exact pose (no re-detection!)
   └─► Returns: object grasps
```

## Detailed Workflow

### Step 1: Detect Load Carrier
```
Service: rc_load_carrier-detect_load_carriers
Purpose: Detect the load carrier position and dimensions
Result: Pose, inner dimensions, outer dimensions
```

**API Endpoint:** `PUT /api/v2/pipelines/0/nodes/rc_load_carrier/services/detect_load_carriers`

**Request:**
```json
{
  "args": {
    "pose_frame": "external",
    "load_carrier_ids": ["lc_generic"]
  }
}
```

**Response:** Returns load carrier pose, inner dimensions, outer dimensions

### Step 2: Set Load Carrier with EXACT_POSE
```
Service: rc_load_carrier_db-set_load_carrier
Purpose: Save the detected pose with pose_type = "EXACT_POSE"
Result: Load carrier stored in database with exact known pose
```

**API Endpoint:** `PUT /api/v2/nodes/rc_load_carrier_db/services/set_load_carrier`

**Request:**
```json
{
  "args": {
    "load_carrier": {
      "id": "lc_exact",
      "pose_type": "EXACT_POSE",
      "pose": { "position": {...}, "orientation": {...} },
      "pose_frame": "external",
      "inner_dimensions": {...},
      "outer_dimensions": {...}
    }
  }
}
```

### Step 3: CADMatch Detection
```
Service: rc_cadmatch-detect_object
Purpose: Detect objects using the exact load carrier
Result: Grasp poses for detected objects
```

**API Endpoint:** `PUT /api/v2/pipelines/0/nodes/rc_cadmatch/services/detect_object`

**Request:**
```json
{
  "args": {
    "template_id": "Template_Name",
    "pose_frame": "external",
    "load_carrier_id": "lc_exact"
  }
}
```

**Response:** Returns detected grasps with poses

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Load carrier not detected | Check STARTPOSE - is entire carrier visible? |
| Wrong carrier ID | Verify ID matches model in rc_cube |
| CADMatch fails | Check template ID and network connection |
| No objects found | Verify objects are in carrier, template is correct |

## When to Re-detect?

The program runs once and stops. To re-detect the load carrier:
- Run the program again (manually or via external trigger)
- Add a loop if continuous operation is needed
- Trigger re-detection after N picks (requires modification)

**When to re-detect:**
- Carrier has been moved or repositioned
- Starting a new batch of objects
- After a configurable number of picks (e.g., every 10 picks)
- On operator request
- After a detection failure (carrier might have shifted)

## Notes

- This is a **minimal example** focused on the three-step workflow
- No pick-and-place functionality (can be added later)
- No loop (runs once then stops)
- Perfect for testing and understanding the workflow
- Ready to extend with your specific application logic