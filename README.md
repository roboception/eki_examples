# <img src="logos/rc_eki.svg" alt="Roboception EKI Logo" height="45" align="right" /> EKI-Examples

This repository provides example applications demonstrating how to integrate [rc_visard](https://roboception.com/rc_visard) 
or [rc_cube](https://roboception.com/product/rc_cube-s/) devices with KUKA robots using the EKI (Ethernet KRL) interface.

These examples are based on KUKA's EthernetKRL technology package. Official documentation: [KUKA EthernetKRL 2.2 Manual](http://supportwop.com/IntegrationRobot/content/6-Syst%C3%A8mes_int%C3%A9grations/Ethernet_KRL/KST_EthernetKRL_22_en.pdf)

## Example Applications

* **CADMatch**: Demonstrates CAD-based object detection and pick-and-place operations
* **ItemPick**: Demonstrates continuous bin-picking for bags. Program loops until bin is empty, detecting items and trying available grasp poses for each detection
* **HandEyeCalibration**: Performs an 8-pose calibration routine to align camera and robot coordinate systems
* **AprilTag**: Implements AprilTag detection and robot positioning
* **Measure**: Demonstrates depth measurement and region analysis capabilities

Each example includes complete robot programs with error handling, integration with respective rc_reason modules, and basic motion sequences.

## ⚠️ Prerequisites

Before running any example:

1. **EKI Configuration**:
   * Download the required EKI XML configuration files from the [rc_cube documentation](https://doc.rc-cube.com/latest/en/eki.html#eki-xml-configuration-files) or [rc_visard documentation](https://doc.rc-visard.com/latest/en/eki.html#eki-xml-configuration-files)
   * Store all EKI XML files in `C:\KRC\ROBOTER\Config\User\Common\EthernetKRL` on the robot controller
   * Configure the IP address of your rc_visard or rc_cube in the EKI XML files
   * Ensure service names in the code match exactly with your EKI XML configuration files

2. **System Requirements**:
   * rc_visard or rc_cube with EKIBridge license
   * KUKA Robot Controller with KUKA.EthernetKRL

For detailed configuration instructions, refer to the [Ethernet connection configuration](https://doc.rc-cube.com/latest/en/eki.html#ethernet-connection-configuration) documentation.

## ⚠️ Safety Notices

* All robot poses in these examples must be adjusted for your specific setup before use
* These applications should only be executed by experienced robot programmers who have thoroughly reviewed and understood the code
* The authors are not liable for any damages that may occur from using these example applications

## Getting Started

1. Review the safety notices and documentation
2. Ensure your system meets the requirements
3. Download and configure the required EKI XML files
4. Select the appropriate example for your application
5. Adjust robot poses and parameters for your setup
6. Test thoroughly in a safe environment

## Error Handling and Troubleshooting

### EKI00023 "Initialization already performed"

If you stop a program mid-execution and restart it, you may encounter error **EKI00023 "Initialization already performed"**. This happens because `EKI_Init()` allocates resources that persist across program stops (but not resets). See [KUKA EthernetKRL documentation](http://supportwop.com/IntegrationRobot/content/6-Syst%C3%A8mes_int%C3%A9grations/Ethernet_KRL/KST_EthernetKRL_22_en.pdf) pages 29-30 for details.

**Solution: Use global flags to track initialization state**

This approach requires an additional PUBLIC .dat file per program, which is not included in these examples for simplicity, but can be easily added for production deployments.

1. Create a separate PUBLIC .dat file for global flags:
```krl
DEFDAT ProgramName_GlobalFlags PUBLIC
   DECL PUBLIC BOOL EKI_DETECT_INITIALIZED
ENDDAT
```

2. In your main program, check the flag before initializing:
```krl
DEF ProgramName()
   DECL GLOBAL BOOL EKI_DETECT_INITIALIZED
   
   EKI_DETECT_INITIALIZED = FALSE  ; Reset at program start
   
   IF NOT EKI_DETECT_INITIALIZED THEN
      RET = EKI_INIT(serviceNameDetect[])
      IF RET.Msg_No == 0 THEN
         EKI_DETECT_INITIALIZED = TRUE
      ENDIF
   ENDIF
END
```

**Why separate .dat files?** KUKA requires all variables in a `DEFDAT ... PUBLIC` block to be declared `PUBLIC`. Keep global flags in their own file to avoid making all program variables global.

**When is this needed?** Production environments where operators may stop/restart programs frequently. Not required if programs always run to completion or manual reset is acceptable.

## Documentation

* [rc_visard EKI Documentation](https://doc.rc-visard.com/latest/en/eki.html)
* [rc_cube EKI Documentation](https://doc.rc-cube.com/latest/en/eki.html)
* [rc_visard Manual](https://doc.rc-visard.com)
* [rc_cube Manual](https://doc.rc-cube.com)

## Support

If you have any questions, technical support contact information can be found at [roboception.com/support](https://roboception.com/en/support/).