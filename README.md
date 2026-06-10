Based on the provided code map, I have sufficient information about this ultrasonic radar project. Let me create a comprehensive README.md:

# 超声波雷达 (Ultrasonic Radar)

基于 ESP32-S3 的超声波雷达测距系统，通过舵机带动超声波传感器进行 360 度扫描，并通过 Web 界面实时显示检测到的障碍物位置和距离。

## 功能特性

- **360度扫描**：通过舵机带动超声波传感器进行全方位扫描
- **实时测距**：使用 HC-SR04 超声波传感器进行精确测距
- **Web 界面**：通过浏览器实时查看雷达图像和数据
- **WiFi AP 模式**：设备作为 WiFi 热点，可直接连接访问
- **WebSocket 通信**：实时推送雷达数据到前端展示

## 硬件连接

### ESP32-S3 与超声波模块

| ESP32-S3 | HC-SR04 |
|----------|---------|
| GPIO 4    | TRIG    |
| GPIO 5    | ECHO   |
| 5V        | VCC    |
| GND       | GND    |

### ECHO 分压接线

由于 ESP32-S3 的 IO 口输入电压为 3.3V，而 HC-SR04 ECHO 输出为 5V，需要使用分压电路：

```
HC-SR04 ECHO ── R1 (10KΩ) ──┬── ESP32 IO (ECHO)
                           │
                           └── R2 (20KΩ) ── GND
```

### 舵机供电说明

- 舵机需要 5V 供电，建议使用独立电源
- 如果使用 ESP32 USB 供电，可能因电流不足导致舵机抖动

## 软件架构

### 数据流

```
┌─────────────┐    ┌──────────┐    ┌────────────┐    ┌───────────┐
│  超声波传感器 │──▶│ 测距模块  │──▶│  雷达任务  │──▶│ WebSocket │
└─────────────┘    └──────────┘    └────────────┘    └───────────┘
                                           │
                                           ▼
                                    ┌───────────┐
                                    │  Web 界面  │
                                    └───────────┘
```

### 模块说明

- **radar.c/h**: 雷达扫描控制，角度范围 0-180°
- **servo.c/h**: 舵机 PWM 控制
- **ultrasonic.c/h**: 超声波测距，测量范围 2cm-400cm
- **web_server.c/h**: HTTP 服务器 + WebSocket
- **wifi_ap.c/h**: WiFi 热点配置 (SSID: RadarAP)

## 编译与烧录

### 环境要求

- ESP-IDF v4.4+
- Python 3.8+
- GNU Make

### 编译步骤

1. **设置 IDF 环境**
   ```bash
   export IDF_PATH=/path/to/esp-idf
   . $IDF_PATH/export.sh
   ```

2. **设置目标芯片**
   ```bash
   cd esp32-web
   idf.py set-target esp32s3
   ```

3. **修改配置（可选）**
   ```bash
   idf.py menuconfig
   ```

4. **编译**
   ```bash
   idf.py build
   ```

5. **烧录并监视串口**
   ```bash
   idf.py flash monitor
   ```

## 使用方法

### 连接设备

1. 设备上电后会自动创建 WiFi 热点
2. 使用手机或电脑连接 WiFi：
   - **SSID**: RadarAP
   - **密码**: 12345678

### 雷达页面

打开浏览器访问 `http://192.168.4.1` 即可查看雷达界面。

### 参数调整

在 Web 界面中可以调整以下参数：

- 扫描速度
- 最小/最大距离
- 舵机角度范围

### WebSocket 数据格式

发送的雷达数据为 JSON 格式：

```json
{
  "angle": 90,
  "distance": 150
}
```