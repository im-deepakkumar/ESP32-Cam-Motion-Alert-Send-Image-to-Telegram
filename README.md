ESP32-CAM Motion Alert System with Telegram Integration

Objectives:
1. To detect motion using the PIR sensor and notify the user remotely.
2. To capture and transmit images using the ESP32-CAM module.
3. To integrate IoT with messaging platforms for real-time surveillance.

Details of Components:
1. ESP32-CAM:
The ESP32-CAM is a low-cost microcontroller module with an integrated OV2640camera and microSD card slot. It supports Wi-Fi and Bluetooth, making it ideal forwireless IoT applications including video streaming and surveillance.

2. PIR Motion Sensor:
A passive infrared sensor that detects motion by sensing changes in infrared radiation. When motion is detected, it outputs a HIGH signal to the ESP32-CAM.

3. TTL Programmer:
Used for uploading code to the ESP32-CAM module via its UART interface. It converts USB signals to serial.

Working of Designed Model:
• When motion is detected by the PIR sensor, it sends a signal to the ESP32-CAM. The camera then initializes, captures a snapshot of the scene, and sends it via Wi-Fi to a
Telegram bot using the bot token and chat ID. This allows remote users to receive realtime photo alerts directly on their phones.
• The system is ideal for low-cost surveillance setups, and can be deployed at doors, garages, or any sensitive areas requiring monitoring..
