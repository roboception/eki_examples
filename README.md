# EKI-Examples

This repository provides example applications demonstrating how to integrate [rc_visard](https://roboception.com/rc_visard) 
or [rc_cube](https://roboception.com/product/rc_cube-s/) devices with KUKA robots using the EKI interface.

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

## Documentation

* [rc_visard EKI Documentation](https://doc.rc-visard.com/latest/en/eki.html)
* [rc_cube EKI Documentation](https://doc.rc-cube.com/latest/en/eki.html)
* [rc_visard Manual](https://doc.rc-visard.com)
* [rc_cube Manual](https://doc.rc-cube.com)


## Support

If you have any questions, technical support contact information can be found at [roboception.com/support](https://roboception.com/en/support/).