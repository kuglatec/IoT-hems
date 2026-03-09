# IoT-hems

A Home Energy Management System (HEMS) built with a **Raspberry Pi** as the central host and **ESP32** microcontrollers as IoT clients. The system supports temperature monitoring, lighting control, and optional camera integration.

---

## Table of Contents

- [Overview](#overview)
- [Hardware Requirements](#hardware-requirements)
- [System Architecture](#system-architecture)
- [Features](#features)
- [Getting Started](#getting-started)
  - [Raspberry Pi Setup](#raspberry-pi-setup)
  - [ESP32 Client Setup](#esp32-client-setup)
- [Project Structure](#project-structure)
- [License](#license)

---

## Overview

IoT-hems connects one or more ESP32 devices to a central Raspberry Pi hub over Wi-Fi. Each ESP32 node can sense its environment (temperature, light levels) and control actuators (LEDs, relays for lighting). An optional camera module can be attached to either the Raspberry Pi or an ESP32-CAM for visual monitoring.

---

## Hardware Requirements

### Host (Raspberry Pi)
- Raspberry Pi 3 / 4 / Zero 2 W (or newer)
- MicroSD card (8 GB or larger, class 10 recommended)
- Power supply (5 V / 3 A)
- Wi-Fi connection (built-in or USB adapter)

### IoT Clients (ESP32)
- ESP32 development board (e.g., ESP32-DevKitC, ESP32-WROOM-32)
- Temperature sensor (e.g., DHT22 / DS18B20 / BME280)
- LED or relay module for lighting control
- *(Optional)* ESP32-CAM module for camera support

---

## System Architecture

```
┌──────────────────────────────────────┐
│           Raspberry Pi (Host)        │
│  - MQTT Broker (e.g. Mosquitto)      │
│  - Dashboard / Web UI                │
│  - Optional: Camera stream           │
└────────────────┬─────────────────────┘
                 │ Wi-Fi / MQTT
      ┌──────────┴──────────┐
      │                     │
┌─────▼──────┐       ┌──────▼─────┐
│  ESP32 #1  │  ...  │  ESP32 #N  │
│ - Temp     │       │ - Temp     │
│ - Lighting │       │ - Lighting │
│ - (Camera) │       │            │
└────────────┘       └────────────┘
```

The Raspberry Pi runs an MQTT broker. Each ESP32 client connects to the broker, publishes sensor readings, and subscribes to control topics for actuators.

---

## Features

| Feature              | Description                                                        |
|----------------------|--------------------------------------------------------------------|
| Temperature Display  | Each ESP32 reads a temperature sensor and publishes readings over MQTT. The Raspberry Pi aggregates and displays values on a dashboard. |
| Lighting Control     | ESP32 nodes control LEDs or relay-switched lights via MQTT commands from the host. |
| Camera (optional)    | An ESP32-CAM or USB/CSI camera on the Raspberry Pi can stream or capture images for monitoring. |

---

## Getting Started

### Raspberry Pi Setup

1. **Install Raspberry Pi OS** (Lite or Desktop) on your SD card using [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

2. **Enable Wi-Fi and SSH** in the imager settings or via `raspi-config`.

3. **Install an MQTT broker** (Mosquitto):
   ```bash
   sudo apt update && sudo apt install -y mosquitto mosquitto-clients
   sudo systemctl enable mosquitto
   sudo systemctl start mosquitto
   ```

4. *(Optional)* **Install a dashboard** such as Node-RED or Home Assistant to visualize sensor data and control devices.

### ESP32 Client Setup

1. Install the [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/).

2. Add ESP32 board support:
   - Arduino IDE: add `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json` in *Preferences → Additional Board Manager URLs*, then install **esp32** via the Board Manager.

3. Install required libraries (via Arduino Library Manager or PlatformIO):
   - `PubSubClient` – MQTT client
   - `DHT sensor library` (or the library matching your sensor)
   - `WiFi` (included with ESP32 Arduino core)

4. Open `src/clients/client.ino`, configure your Wi-Fi credentials and the Raspberry Pi's IP address, then flash to the ESP32.

5. After flashing, the ESP32 will:
   - Connect to Wi-Fi
   - Connect to the MQTT broker on the Raspberry Pi
   - Publish temperature readings to `hems/<device_id>/temperature`
   - Subscribe to `hems/<device_id>/light` for lighting commands (`ON` / `OFF`)

---

## Project Structure

```
IoT-hems/
├── src/
│   └── clients/
│       └── client.ino   # Arduino sketch for ESP32 IoT clients
├── LICENSE
└── README.md
```

---

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file included in this repository.
