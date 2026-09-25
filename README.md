# ESP32 Universal Smart IR Remote

A standalone, web-controlled universal infrared (IR) remote built on the ESP32 platform. The system operates as an independent WiFi Access Point hosting an embedded web interface, allowing users to capture, map, save, and transmit IR commands for consumer appliances (televisions, set-top boxes, soundbars) directly from any mobile or desktop web browser—no external router, cloud infrastructure, or native apps required.

---

## Features

* **Protocol Agnostic IR Learning:** Leverages a 38 kHz demodulating receiver to parse and identify pulse trains (NEC, Sony, RC5/RC6, Samsung, etc.), capturing protocol type, payload hex, and bit length.
* **Persistent Key Mapping:** Uses the ESP32's non-volatile storage (`Preferences` API) to store mapped button names and IR signal definitions across resets and power cycles.
* **Standalone Access Point:** Broadcasts its own localized WiFi network (`SmartRemote_ESP32`) serving an asynchronous, single-page web app.
* **Adaptive Remote Interface:** Renders a 3-column circular keypad grid mimicking handheld physical remotes, complete with a live size-scaling slider.
* **Transistor-Boosted Blaster:** Incorporates a low-side NPN driver circuit (BC547 / 2N2222) to deliver sufficient burst current to the IR LED for room-wide transmission coverage.

---

## Hardware Architecture & Bill of Materials

| Component | Quantity | Description / Value |
| --- | --- | --- |
| **ESP32 Development Board** | 1 | NodeMCU ESP-WROOM-32 or equivalent |
| **IR Receiver Module** | 1 | 38 kHz Demodulating Receiver (e.g., VS1838B, TSOP38238) |
| **IR Transmitter LED** | 1 | 940 nm or 850 nm Clear IR Emitter (5 mm) |
| **NPN Transistor** | 1 | BC547 (up to 100 mA) or 2N2222 (up to 600–800 mA) |
| **Base Resistor** | 1 | 1 kΩ (limits GPIO base current) |
| **LED Current-Limiting Resistor** | 1 | 47 Ω – 100 Ω (or two 220 Ω in parallel for direct testing) |
| **Breadboard & Jumper Wires** | — | Prototyping interconnects |

---

## Circuit Schematic & Pinout

### 1. IR Receiver (VS1838B)

Facing the curved sensor dome with pins pointing downwards:

* **Left Pin (OUT / Signal):** Connect to `GPIO 15`
* **Middle Pin (GND):** Connect to ESP32 `GND`
* **Right Pin (VCC):** Connect to ESP32 `3.3V`

### 2. High-Current IR Transmitter Circuit (NPN Driver)

* **Base (Middle Pin of BC547):** Connect to ESP32 `GPIO 4` via a **1 kΩ resistor**.
* **Emitter (Right Pin of BC547):** Connect directly to ESP32 `GND`.
* **Collector (Left Pin of BC547):** Connect to the **Cathode** (short leg / flat edge) of the IR LED.
* **IR LED Anode (long leg):** Connect to ESP32 `VIN` (5V) via a **47 Ω resistor**.

```text
                  +5V (VIN)
                     |
                   [47Ω]
                     |
                 (Anode)
                [ IR LED ]
                (Cathode)
                     |
                     +---- Collector (Pin 1)
                                |
GPIO 4 ---[ 1kΩ ]---------- Base (Pin 2)  [BC547]
                                |
                           Emitter (Pin 3)
                                |
                               GND

```

> **Warning:** Never connect the IR emitter directly to 5V without a current-limiting resistor; doing so causes immediate thermal runaway and destroys both the LED and the transistor.

---

## Software & Library Setup

1. Install the latest version of the [Arduino IDE](https://www.arduino.cc/en/software?utm_source=gemini).
2. Install the ESP32 Board Package via Board Manager:
* URL: `[https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json](https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json)`


3. Install the required dependency via **Sketch > Include Library > Manage Libraries...**:
* Search for **`IRremoteESP8266`** (by David Conran, Ken Shirriff, et al.) and click **Install**.


4. Select your board (`DOIT ESP32 DEVKIT V1` or appropriate generic ESP32 module).
5. Open the project `.ino` sketch, verify, and upload via USB.

---

## Operational Guide

### 1. Connection

* Power the ESP32 via standard 5V micro-USB / USB-C.
* On your phone or laptop, join the WiFi network:
* **SSID:** `SmartRemote_ESP32`
* **Password:** `password123`


* Open a browser and navigate to: `[http://192.168.4.1](http://192.168.4.1)`

### 2. Learning Mode (Registering Keys)

1. Select target appliance profile (**Television** or **Set-Top Box**).
2. Tap **Switch to Learn Mode**.
3. Direct your original handheld remote at the receiver module (GPIO 15) and depress the desired key.
4. The web dashboard will update with the detected protocol and hex value.
5. Enter a key alias (e.g., `Power`, `Vol+`, `CH1`) and click **Save Button**.
6. The key payload is committed to non-volatile flash.

### 3. Remote Mode (Transmission)

1. Tap **Switch to Remote Mode**.
2. Dynamically adjust key diameters using the **Adjust Button Size** slider.
3. Tap any rendered button; the ESP32 reads the parameter set from NVS flash and pulses the IR transmitter on GPIO 4 at 38 kHz.
