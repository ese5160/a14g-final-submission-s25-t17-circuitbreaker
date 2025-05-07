[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/AlBFWSQg)
# a14g-final-submission

    * Team Number: 17
    * Team Name: CircuitBreaker
    * Team Members: Yuetian Zhao&Shuowen Gu
    * Github Repository URL: https://github.com/ese5160/a14g-final-submission-s25-t17-circuitbreaker
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
Below is a review of each hardware and software requirement from our project specification, including how we validated performance, whether the requirement was met, and relevant data.

| **Requirement**                  | **Target**               | **Test Method**                                                                 | **Result**                    | **Met?** |
|----------------------------------|---------------------------|----------------------------------------------------------------------------------|-------------------------------|----------|
| UART Communication Rate         | 115200 bps                | Measured UART TX from MCU to PC using a logic analyzer. Verified baud rate.     | 115211 bps (error: +0.01%)    | ✅ Yes   |
| I2C Communication Rate          | 100 kHz                   | Used logic analyzer to capture SCL frequency between MCU and IMU.                | 98.7 kHz (±1.3%)              | ✅ Yes   |
| IMU Data Accuracy               | Accel ±16g, Gyro ±2000°/s | Compared raw sensor data to known motion (manual swing + phone IMU baseline).    | Within ~5% drift over 5s      | ⚠️ Partial |
| IMU Data Sampling Period        | 200 Hz target (5ms)       | Captured timestamped data samples via UART and computed average interval.        | ~5.03 ms/sample               | ✅ Yes   |
| UART Data Integrity             | No data corruption        | Sent known pattern (0xAA, 0x55, etc.) repeatedly and checked on receiver.        | 0 errors in 1000 packets      | ✅ Yes   |

---

### IMU Validation Details

We validated the IMU data by manually moving the device to the right for approximately 5 seconds and collecting real-time readings from the accelerometer and gyroscope. A comparison was made against expected motion trends and a smartphone IMU (Google Science Journal app) used as a baseline.

#### Sample Data (Rightward Motion)

| Sample | Accel X | Accel Y | Accel Z | Gyro X | Gyro Y | Gyro Z |
|--------|---------|---------|---------|--------|--------|--------|
| 1      | 88      | 727     | 2314    | -695   | -276   | 41     |
| 2      | 69      | 749     | 2138    | -186   | -479   | 189    |
| 3      | 90      | 682     | 1797    | 336    | 58     | 168    |
| 4      | 98      | 608     | 2065    | -216   | -479   | 67     |
| 5      | 333     | 604     | 2123    | -1009  | -1054  | -78    |
| 6      | 174     | 620     | 1952    | -530   | -516   | -52    |

These values were collected at approximately 5 ms intervals using a FreeRTOS task. The motion was consistent with expected acceleration in the X-axis and significant angular velocity primarily in the X and Y axes.

#### Validation Methodology

- **Baseline**: We used a smartphone IMU as a qualitative reference.
- **Motion Profile**: A smooth rightward motion was performed for ~5 seconds.
- **Consistency**: We checked if accelerometer X values and gyro X/Y values changed consistently with motion direction.
- **Sampling Rate**: Verified average time between samples using timestamps and UART printouts — measured ~5.03 ms/sample (std dev < 0.2 ms).
- **Drift & Bias**: Gyro drift over time was noted (~2–3°/s on Z), which is within expected bounds without software compensation.

#### Summary

The IMU measurements were consistent and within acceptable error for gesture recognition. Although some drift was present in the gyroscope, the sensor was not calibrated beyond factory settings. Future work could include:
- Applying offset/bias compensation
- Performing a 6-point calibration
- Validating against a more precise IMU or motion capture system

> Overall, the IMU performance meets the requirements of our application, though not perfectly accurate in absolute terms.

## 4. Project Photos & Screenshots

## Codebase
- A link to your final embedded C firmware codebases: 
- A link to your Node-RED dashboard code: http://20.83.165.177:1880/#flow/f0b6c745df1e85c5
- Links to any other software required for the functionality of your device: 

