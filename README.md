[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/AlBFWSQg)
# a14g-final-submission

    * Team Number: 
    * Team Name: 
    * Team Members: 
    * Github Repository URL: 
    * Description of test hardware: (development boards, sensors, actuators, laptop + OS, etc) 

## 1. Video Presentation
https://drive.google.com/file/d/1Q8Uip4zGH3cBUoYxHP9DGPIWs58QoFy_/view?usp=sharing
https://drive.google.com/file/d/13tFrA57lfs7_Pe5DuiQjUAeWHU28Ekg5/view?usp=sharing

## 2. Project Summary
### Device Description
The Magic Wand IoT Device is a handheld, gesture-controlled interface that lets users perform smart-home actions with intuitive “spell” motions. Inspired by the desire to make everyday home automation more playful and accessible, it solves the problem of complex multi-app control by providing a single, gesture-driven tool. The wand leverages Wi-Fi and MQTT to send commands to cloud or local hubs, enabling real-time control of lights, music, and more.

### Device Functionality
- **Motion Sensing & Gesture Mapping**  
  - **Sensor:** ICM-42670-P 6-axis IMU (I²C) samples accelerometer & gyroscope at ≥100 Hz.  
  - **Firmware:** Onboard SAMW25 decodes raw data into predefined gestures (e.g., “Triangle,” “Circle”).  
  - **MQTT Publish:** Gesture → JSON payload `{ "gesture": "Circle"}` on `wand/gesture` topic.
  ![alt text](<system-level block diagram-1.png>)
  ![alt text](<system-level block diagram-2.png>)
- **Fingerprint Authentication**  
  - **Module:** BM-Lite capacitive sensor (SPI/UART) with built-in matching.  
  - **Flow:** On power-up, smart device won;r be able to receive any MQTT message until valid `{ "user_id": 2 }` is published on `wand/auth/request`; BM-Lite replies on `wand/auth/response`.

- **Feedback Actuators**  
  - **RGB LED:** Signals Wi-Fi, auth, and action status via color codes.  
  - **Vibration Motor:** Haptic confirmation on gesture recognition or errors.

- **Cloud Integration**  
  - **Broker:** Home Assistant local broker and online Microsoft Azure MQTT Broker.  
  - **Command Processing:** Cloud subscribes to `wand/gesture`; maps gestures → home-automation actions; then publishes control messages on topics like `home/lights/bedroom/set` with payload `{ "state": "ON" }`.

### Challenges
- **Sensor Calibration & Noise Filtering**  
  - **Issue:** Raw IMU data was noisy, causing false positives.  
  - **Solution:** Implemented a complementary filter + sliding-window smoothing in firmware to stabilize readings.

- **Reliable MQTT Connectivity**  
  - **Issue:** SAMW25 occasional dropouts in poor Wi-Fi environments.  
  - **Solution:** Added auto-reconnect logic with exponential backoff and local caching of unsent messages in FRAM.

- **Fingerprint Module Integration**  
  - **Issue:** BM-Lite library conflicts with I²C driver.  
  - **Solution:** Refactored bus routines to multiplex SPI/UART and reorganized interrupt priorities to avoid contention.

### Prototype Learnings
- **Modular Firmware Architecture:** Splitting sensor, auth, and comms into isolated threads simplified debugging and OTA patching.  
- **Thread Communication:** Used FreeRTOS queues for passing gesture events from the IMU thread to the MQTT thread, ensuring non-blocking operation.  
- **Power Management Trade-offs:** Aggressive sleep modes lowered power but introduced wake-latency; next iteration will tune wake thresholds.

If rebuilding:
- Adopt an RTOS-agnostic HAL to ease migration to other MCUs.  
- Integrate external EEPROM for larger log storage when offline.

### Next Steps & Takeaways
- **Enhancements:**  
  - Add “scene” gestures that trigger multiple devices in a sequence.  
  - Implement end-to-end encryption (TLS over MQTT) for secure command delivery.  
  - Build a web dashboard for live gesture analytics and remote device enrollment.

- **Course Insights (ESE5160):**  
  - Learned the value of a well-structured MQTT topic hierarchy and QoS settings for robust IoT communication.  
  - Gained hands-on experience with Node-RED for rapid prototyping of cloud workflows and device dashboards.  
  - Understood trade-offs in low-power embedded design, from peripheral sleep to interrupt-driven wake-ups.

### Project Links
 - NodeRed UI: http://20.83.165.177:1880/ui/#!/0?socketid=6hr5YcUWveSmIbtoAAC3
 - Final PCBA: https://upenn-eselabs.365.altium.com/designs/76787DB1-9A10-4CC7-A173-6C2606C8EC13?variant=[No+Variations]&activeDocumentId=CircuitBreaker.PcbDoc&activeView=3D#design


## 3. Hardware & Software Requirements


## 4. Project Photos & Screenshots

## Codebase

- A link to your final embedded C firmware codebases
- A link to your Node-RED dashboard code
- Links to any other software required for the functionality of your device

