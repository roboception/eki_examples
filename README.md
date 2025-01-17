EKI-Examples
============

This repository provides example applications demonstrating how to integrate [Roboception](https://roboception.com)'s [rc_visard](https://roboception.com/rc_visard) 
or [rc_cube](https://roboception.com/product/rc_cube-s/) with KUKA robots using the EKI interface.

Example Applications
------------------
* **CADMatch**: Demonstrates CAD-based object detection and pick-and-place operations
* **ItemPick**: Demonstrates continuous bin-picking for bags. Program loops until bin is empty, detecting items and trying available grasp poses for each detection.
* **HandEyeCalibration**: Performs an 8-pose calibration routine to align camera and robot coordinate systems
* **AprilTag**: Implements AprilTagdetection and robot positioning
* **Measure**: Demonstrates depth measurement and region analysis capabilities

Each example includes:
* Complete robot program with error handling
* Integration with respective rc_reason modules
* Basic motion sequences

⚠️ Safety Notices
----------------
* All robot poses in these examples must be adjusted for your specific setup before use
* These applications should only be executed by experienced robot programmers who have thoroughly reviewed and understood the code
* Roboception is not liable for any damages that may occur from using these example applications

System Requirements
-----------------
* rc_visard or rc_cube with EKIBridge license
* KUKA Robot Controller with KUKA.EthernetKRL

Documentation
------------
Comprehensive documentation is available at:
* [rc_visard EKI Documentation](https://doc.rc-visard.com/latest/en/eki.html)
* [rc_cube EKI Documentation](https://doc.rc-cube.com/latest/en/eki.html)
* [rc_visard Manual](https://doc.rc-visard.com)
* [rc_cube Manual](https://doc.rc-cube.com)

For more information about Roboception and our product portfolio, please visit [roboception.com](https://www.roboception.com)

Getting Started
-------------
1. Review the safety notices and documentation
2. Ensure your system meets the requirements
3. Select the appropriate example for your application
4. Adjust robot poses and parameters for your setup
5. Test thoroughly in a safe environment

Support
-------
If you have any questions, our technical support team is here to help. Contact information can be found at [roboception.com/support](https://roboception.com/en/support/).