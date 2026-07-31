---
title: "Hardware Integration"
date: "2026-07-28"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Overview and objectives

YOLO UNO is the primary device for this workshop. It reads DHT20 temperature/humidity and a raw analog light value, controls fan, light/relay, and curtain servo outputs, sends telemetry, polls a pending command, executes it once, and sends ACK.

## Introduction to YOLO UNO

YOLO UNO is a development board designed with the form factor and pin layout of the traditional Arduino Uno while using the **ESP32-S3** microcontroller from Espressif. The board provides **16 MB of Flash memory** and up to **8 MB of PSRAM**, making it suitable for embedded, IoT, and on-device data-processing applications.

In addition to its Arduino Uno-compatible size and pin layout, YOLO UNO includes **12 integrated Grove ports**. These connectors simplify the process of attaching sensors and actuators without requiring a breadboard or multiple individual jumper wires.

In the **AWS IoT Monitoring and Control Dashboard** project, YOLO UNO:

- reads temperature and humidity from the DHT20 sensor;
- reads raw ADC values from the analog light sensor;
- controls the two-pin fan module;
- controls the LED or relay light module;
- controls the curtain servo;
- displays system status on an LCD 1602 I2C;
- connects to Wi-Fi and sends telemetry to the FastAPI backend; and
- polls the backend for commands, executes them, and sends acknowledgements.

Most sensors and actuators are connected through the Grove ports in the center of the board. The curtain servo uses the three-pin **D11 GVS port**, which is mapped to **GPIO38** in the firmware.

The physical prototype below shows the YOLO UNO and the connected display, sensors, fan, and curtain-control servo used by the project.

![YOLO UNO hardware prototype with sensors and actuators](/images/5-Workshop/5.6-hardware/yolo-uno-hardware-setup.png)
*Figure 10. The physical hardware prototype consisting of YOLO UNO, an LCD display, temperature and humidity sensing, a light sensor, a fan, and a curtain-control servo.*

> **Note:** This image was extracted from a video, so some details may appear slightly blurred. See the [full demonstration video on Google Drive](https://drive.google.com/file/d/1T97dUY58hbT2ppxvg7ESR12Jg9BA828W/view?usp=sharing) for a clearer view.

## Step 1 - Wire the source-defined hardware

The active `hardware/src/main.cpp` defines this map; it takes precedence over stale prose elsewhere in the source repository:

The YOLO UNO pinout image has not yet been provided. Do not treat a pinout as a complete wiring diagram; use the verified table below together with the firmware definitions.

<!-- TODO: Add yolo-uno-pinout-gpio-mapping.png -->

### Hardware Port Mapping

| Device | YOLO UNO Physical Port | Firmware Pin |
| :--- | :--- | :--- |
| Analog light sensor | Grove `A1-A0` | `A0 / GPIO1` |
| Two-control-pin fan | Grove `D8-D7` | `D8 / GPIO17`, `D7 / GPIO10` |
| LED or relay light | Grove `D4-D3` | `D3 / GPIO6` |
| Curtain servo | `D11` GVS port | `D11 / GPIO38` |
| DHT20 | Grove `I2C1` | `SDA / GPIO11`, `SCL / GPIO12` |
| LCD 1602 I2C | Grove `I2C2` | `SDA / GPIO11`, `SCL / GPIO12` |

DHT20 is connected to Grove `I2C1`, while LCD 1602 uses Grove `I2C2`. Both ports share SDA/GPIO11 and SCL/GPIO12 on the same I2C bus. DHT20 uses address `0x38`; the firmware auto-detects the LCD at `0x21`, `0x27`, or `0x3F`. If the LCD requires 5 V, use a bidirectional logic-level converter and never connect a 5 V pull-up directly to an ESP32-S3 GPIO. The modules share the I2C bus; they are not wired directly to each other.

The firmware does not include PIR, ultrasonic sensors, buzzers, or MQTT. Use a suitable actuator supply and a common ground; never draw fan or servo current directly from a GPIO.

## Step 2 - Prepare PlatformIO

Open the hardware project in VS Code. Confirm:

- the YOLO UNO / ESP32-S3 board JSON exists and is referenced correctly;
- `platformio.ini` selects the intended environment and libraries;
- Serial Monitor baud is `115200`;
- the environment is `yolo_uno` on ESP32-S3;
- ArduinoJson, ESP32Servo, DHT20, and LiquidCrystal_I2C dependencies resolve;
- `include/secrets.example.h` is committed; and
- `include/secrets.h` is local and ignored.

Use this secret shape:

```cpp
#pragma once
constexpr char WIFI_SSID[] = "<YOUR_WIFI_SSID>";
constexpr char WIFI_PASSWORD[] = "<YOUR_WIFI_PASSWORD>";
constexpr char API_BASE_URL[] = "http://<ALB_DNS_NAME>";
constexpr char DEVICE_ID[] = "room_01";
```

Never publish the real Wi-Fi password. The verified device route is direct HTTP to the ALB DNS name; do not place an EC2 instance IP or the CloudFront frontend domain in the firmware unless a separately tested device route is introduced.

## Step 3 - Validate sensors and actuators locally

1. Initialize I2C and confirm the DHT20 responds.
2. Read temperature and humidity; reject invalid/NaN readings.
3. Read and report the **raw analog light value** until the source contains a calibrated conversion.
4. Initialize fan and light outputs to a safe state.
5. Attach the servo and test close at 0° and open at 90°.
6. Confirm the LCD is detected at one of the three supported addresses.
7. Confirm a failed sensor does not repeatedly reset or block the command loop.

Use a driver and flyback protection for inductive loads where required. Servo/fan current must not be drawn directly from a GPIO.

## Step 4 - Send telemetry

The firmware serializes the exact camelCase aliases accepted by the backend:

```json
{
  "deviceId": "room_01",
  "temperature": 25.0,
  "humidity": 60.0,
  "lightIntensity": 512,
  "fan": false,
  "light": false,
  "curtain": false
}
```

`lightIntensity` is a raw analog value. The source sends telemetry every 5000 ms, polls commands every 2000 ms, updates the LCD every 2000 ms, retries Wi-Fi every 10000 ms with a 20000 ms connection timeout, and uses a 7000 ms HTTP timeout.

```text
YOLO UNO read → JSON serialize → POST /api/telemetry → check HTTP status → wait configured interval
```

The ALB forwards these device requests to a Healthy FastAPI target on port 8000. The device does not select or depend on a specific ASG instance.

On non-success status, log the response and retry later with bounded delays. Do not block actuator safety logic indefinitely.

## Step 5 - Poll, execute, and acknowledge commands

Poll:

```text
GET /api/devices/room_01/commands/latest
```

Accept the eight firmware commands: `MODE_AUTO`, `MODE_MANUAL`, `FAN_ON`, `FAN_OFF`, `LIGHT_ON`, `LIGHT_OFF`, `CURTAIN_OPEN`, and `CURTAIN_CLOSE`. A direct actuator command switches the firmware to manual mode. In auto mode, the source turns the fan on at temperature ≥30°C, turns the light on when the analog value is <350, and opens the curtain when the analog value is <700.

```cpp
if (pending && commandId != lastExecutedCommandId) {
  const bool applied = applySupportedCommand(command);
  if (applied) {
    lastExecutedCommandId = commandId;
    sendAck(commandId);
  }
}
```

ACK:

```text
POST /api/devices/room_01/commands/{command_id}/ack
```

If ACK fails after the actuator succeeds, retry the ACK without applying the actuator again. The source stores `autoMode`, `lastAck`, and `pendingAck` in ESP32 Preferences so an ACK can be retried and the same command is not re-executed after reboot. Unsupported commands are rejected by firmware and are not acknowledged.

## Step 6 - Build, upload, and monitor

In a PlatformIO terminal:

```powershell
pio run -e yolo_uno
```

Building the firmware does not require the board to be connected through USB. A successful build produces `firmware.bin` and ends with:

```text
SUCCESS
```

The screenshot below shows that the firmware compiled successfully.

![Successful YOLO UNO firmware build with PlatformIO](/images/5-Workshop/5.6-hardware/platformio-firmware-build.png)
*Figure 12. The YOLO UNO firmware was successfully compiled with PlatformIO using the `yolo_uno` environment, producing `firmware.bin` with a `SUCCESS` result.*

Firmware upload and Serial Monitor access require the YOLO UNO board to be connected to the computer:

```powershell
pio run --target upload
pio device monitor --baud 115200
```

When the board is connected and running, an expected Serial Monitor sequence is:

```text
[wifi] connected
[telemetry] HTTP success for room_01
[command] pending command received: <COMMAND_ID> <SUPPORTED_COMMAND>
[actuator] command applied once
[ack] command acknowledged
```

## Expected Result

PlatformIO compiles the `yolo_uno` environment successfully and creates `firmware.bin`. With the board connected and the remaining integration checks completed, YOLO UNO reads DHT20 and the analog light sensor, updates LCD1602, posts the exact telemetry schema, executes each supported command once, and changes the matching backend command from `Pending` to `Executed` through ACK. Serial evidence must not contain a Wi-Fi password or an unredacted public endpoint.

## Troubleshooting

| Symptom | Check |
| :--- | :--- |
| DHT20 not found | SDA/SCL, address, power, I2C initialization |
| Light value stuck at min/max | ADC pin capability, voltage range, wiring |
| Board resets on actuator action | External supply, current, flyback protection, common ground |
| Wi-Fi reconnect loop | SSID/password, signal, blocking delay, reconnect backoff |
| HTTP timeout | ALB DNS, Wi-Fi/DNS, listener HTTP:80, target health, backend SG, Uvicorn bind |
| Command repeats | Compare command IDs and separate actuator execution from ACK retry |
| Command stays `Pending` | ACK URL/ID/body, HTTP response, backend log |
| Unsupported command | Log and reject it; do not ACK it as executed |

Next: [connect the React dashboard](../5.7-Frontend-Integration/).
