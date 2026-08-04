# ADAS Feature Survey

ere
for real-time applications like Adaptive Cruise Control (ACC) and Forward Collision Warning (FCW), using a bare-metal embedded system like STM32 is often more appropriate due to its real-time capabilities and low latency. However, Raspberry Pi can still be highly valuable for other ADAS functionalities.
- **Lane Departure Warning (LDW):**
    
    - **Function:** Alerts the driver if the vehicle begins to drift out of its lane without signaling.
    - **Raspberry Pi Role:** Process camera feed to analyze lane markings and determine lane departure. Raspberry Pi can handle image processing and machine learning tasks.
- **Traffic Sign Recognition (TSR):**
    
    - **Function:** Detects and interprets traffic signs to inform the driver of speed limits, warnings, and other important signs.
    - **Raspberry Pi Role:** Use camera inputs and image recognition algorithms to identify traffic signs and display relevant information to the driver.
- **Driver Monitoring System (DMS):**
    
    - **Function:** Monitors driver’s alertness and behavior to prevent accidents due to drowsiness or distraction.
    - **Raspberry Pi Role:** Process video from a camera focused on the driver’s face to detect signs of drowsiness or distraction using computer vision techniques.
- **Surround View Monitoring:**
    
    - **Function:** Provides a 360-degree view around the vehicle using multiple cameras.
    - **Raspberry Pi Role:** Integrate and process images from multiple cameras to create a composite view. Useful for parking assistance and low-speed maneuvers.
- **Infotainment Systems:**
    
    - **Function:** Provides multimedia, navigation, and connectivity features within the vehicle.
    - **Raspberry Pi Role:** Serve as a powerful multimedia platform for managing navigation, media playback, and connectivity.
- **Remote Vehicle Diagnostics:**
    
    - **Function:** Monitors vehicle health and performance remotely.
    - **Raspberry Pi Role:** Collect and process diagnostic data from various sensors and systems. Can be used to send diagnostic information to a central server or mobile app.

[

# Rear cross traffic alert (RCTA)

https://www.youtube.com/watch?v=WjhTyOIxY88
https://www.youtube.com/watch?v=aFgMODWrlxY

Important
https://www.hackster.io/Aasai/lane-following-robot-using-opencv-da3d45