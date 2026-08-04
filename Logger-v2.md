### **Logger Overview**
The logger is a crucial component of the ADAS (Advanced Driver Assistance System) in the smart car architecture. It operates as a separate sub-system using a Cortex-M ARM controller, responsible for monitoring, detecting, and reporting any failures in the various ADAS subsystems, such as Adaptive Cruise Control (ACC), Lane Keeping Assist (LKA), Forward Collision Warning (FCW), and Intersection Movement Assist (IMV). 


### **Logger Responsibilities:**
1. **Monitor Sub-system Failures**: The logger continuously monitors the health of all ADAS subsystems (e.g., ACC, LKA, IMV, FCW) to detect malfunctions, anomalies, or failures in real-time.

2. **Report to Main System**: Upon detecting any subsystem failure, the logger immediately sends an error report to the main system for display on the dashboard and to notify the driver.

3. **Log Data**: The logger saves detailed information about subsystem failures in its internal database. This data can be used for future diagnostics, analysis, and trend tracking to improve system reliability.

4. **Upload Data to Cloud**: The logger can periodically upload logged data to a secure cloud storage or diagnostic server. This facilitates remote monitoring, fleet management, or advanced data analytics, enabling over-the-air (OTA) diagnostics and updates to enhance system performance.

### **Subsystem Responsibilities:**

- **ACC (Adaptive Cruise Control) Diagnostic Responsibilities:**
  - Monitor sensor data for anomalies.
  - Detect and log control issues, such as failure to maintain speed, and report critical errors to the logger.

- **LKA (Lane Keeping Assist) Diagnostic Responsibilities:**
  - Monitor lane detection algorithms for failures.
  - Ensure proper functioning of steering actuators and log any calibration or actuator errors.

- **IMV (Intersection Movement Assist) Diagnostic Responsibilities:**
  - Check V2V (Vehicle-to-Vehicle) communication status.
  - Detect and log sensor malfunctions (e.g., radar or lidar) and report any errors.

- **FCW (Forward Collision Warning) Diagnostic Responsibilities:**
  - Monitor forward sensors and ensure proper functionality in collision detection algorithms.
  - Log any processing delays or inaccuracies in collision prediction and report them to the logger.
- **Night Vision Diagnostic Responsibilities:**
  - Report any issues to the logger for storage and alerting the main system.
  - Detect and log failures in the infrared camera or image processing algorithms.
