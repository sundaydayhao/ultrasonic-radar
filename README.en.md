# Ultrasonic Radar

An ultrasonic radar distance measurement system based on ESP32-S3, which rotates an ultrasonic sensor via a servo motor for 360-degree scanning and displays detected obstacle positions and distances in real time through a web interface.

## Features

- **360-Degree Scanning**: Rotates the ultrasonic sensor via servo motor for full-range scanning
- **Real-Time Distance Measurement**: Uses HC-SR04 ultrasonic sensor for precise distance detection
- **Web Interface**: View radar visuals and data in real time via a web browser
- **WiFi AP Mode**: Device acts as a WiFi hotspot for direct connection and access
- **WebSocket Communication**: Streams radar data in real time to the frontend for display

## Hardware Connections

### ESP32-S3 to Ultrasonic Module

| ESP32-S3 | HC-SR04 |
|----------|---------|
| GPIO 4   | TRIG    |
| GPIO 5   | ECHO    |
| 5V       | VCC     |
| GND      | GND     |

### ECHO Voltage Divider Wiring

Since the ESP32-S3 IO input voltage is 3.3V, but the HC-SR04 ECHO output is 5V, a voltage divider circuit is required:

```
HC-SR04 ECHO ── R1 (10KΩ) ──┬── ESP32 IO (ECHO)
                           │
                           └── R2 (20KΩ) ── GND
```

### Servo Power Supply Note

- The servo requires 5V power; an independent power source is recommended
- If powered via ESP32 USB, servo jitter may occur due to insufficient current

## Software Architecture

### Data Flow

```
┌─────────────┐    ┌──────────┐    ┌────────────┐    ┌───────────┐
│ Ultrasonic  │──▶│ Distance │──▶│  Radar     │──▶│ WebSocket │
│ Sensor      │    │ Module   │    │ Task       │    │           │
└─────────────┘    └──────────┘    └────────────┘    └───────────┘
                                           │
                                           ▼
                                    ┌───────────┐
                                    │  Web UI   │
                                    └───────────┘
```

### Module Descriptions

- **radar.c/h**: Radar scanning control, angle range 0–180°
- **servo.c/h**: Servo PWM control
- **ultrasonic.c/h**: Ultrasonic distance measurement, range 2cm–400cm
- **web_server.c/h**: HTTP server + WebSocket
- **wifi_ap.c/h**: WiFi hotspot configuration (SSID: RadarAP)

## Compilation and Flashing

### Requirements

- ESP-IDF v4.4+
- Python 3.8+
- GNU Make

### Compilation Steps

1. **Set up IDF environment**
   ```bash
   export IDF_PATH=/path/to/esp-idf
   . $IDF_PATH/export.sh
   ```

2. **Set target chip**
   ```bash
   cd esp32-web
   idf.py set-target esp32s3
   ```

3. **Modify configuration (optional)**
   ```bash
   idf.py menuconfig
   ```

4. **Build**
   ```bash
   idf.py build
   ```

5. **Flash and monitor serial output**
   ```bash
   idf.py flash monitor
   ```

## Usage

### Connecting to the Device

1. After powering on, the device automatically creates a WiFi hotspot
2. Connect your phone or computer to WiFi:
   - **SSID**: RadarAP
   - **Password**: 12345678

### Radar Interface

Open a browser and navigate to `http://192.168.4.1` to view the radar interface.

### Parameter Adjustment

The web interface allows adjustment of the following parameters:

- Scan speed
- Minimum/maximum distance
- Servo angle range

### WebSocket Data Format

Radar data is transmitted in JSON format:

```json
{
  "angle": 90,
  "distance": 150
}
```