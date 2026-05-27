# Lab Tutorial: Intelligent Smart Lighting Node with ESP32, MQTT, and Voice Control

This repository contains the step-by-step lab guide to building a conditional smart lighting system using an ESP32, physical sensors, an MQTT broker, and local Natural Language Processing (NLP) voice pipelines.

---

## 🛠️ Hardware Component Checklist
* 1x ESP32 Dev Kit (NodeMCU-32S)
* 1x HC-SR501 PIR Motion Sensor Module (5V compliant)
* 1x Analog LDR Light Sensor Module (with AO output)
* 1x Standard LED (Any color)
* 1x 220Ω or 330Ω Resistor
* 1x Half-size or Full-size Breadboard
* Male-to-Male (M-M) and Female-to-Male (F-M) Jumper Wires

---

## 🚀 Step 1: Physical Hardware & Breadboard Wiring

### ⚠️ Core Hardware Rule: Dual-Voltage Safety
The ESP32 microcontroller operates at a **3.3V internal logic level**. However, the PIR sensor requires **5V** to operate its internal circuit, while the Analog Light Sensor sends variable voltage signals to the ESP32’s sensitive Analog-to-Digital Converter (ADC) pins. 

To prevent permanent chip damage, you **must separate the power rails** on your breadboard:
* **Left Side Rails:** Dedicated strictly to **5V constant power**.
* **Right Side Rails:** Dedicated strictly to **3.3V constant power**.
* **Common Ground:** Both sides must share a unified negative path.

### 🔌 Wiring Instructions

1. **Mount the Microcontroller:** Straddle the ESP32 down the center trench of the breadboard so the left and right rows of pins remain isolated.
2. **Establish the Power Buses:**
   * Run a jumper wire from the ESP32 **5V (or VIN)** pin to the **Left Positive (+) rail**.
   * Run a jumper wire from the ESP32 **3.3V** pin to the **Right Positive (+) rail**.
3. **Establish a Unified Ground:**
   * Run a jumper wire from an ESP32 **GND** pin to the **Left Negative (-) rail**.
   * Connect a short jumper wire bridging the **Left Negative (-) rail** directly across to the **Right Negative (-) rail**.
4. **Wire the PIR Motion Sensor:**
   * Connect **VCC** to the Left 5V (+) rail.
   * Connect **GND** to the Left GND (-) rail.
   * Connect **OUT** (Signal) to ESP32 **GPIO27**.
5. **Wire the Analog Light Sensor Module:**
   * Connect **VCC** to the Right 3.3V (+) rail.
   * Connect **GND** to the Right GND (-) rail.
   * Connect **AO** (Analog Output) to ESP32 **GPIO34**.
6. **Wire the Output Testing LED:**
   * Insert the LED into the numbered middle terminal rows. Ensure the legs are in separate rows (e.g., Row 5 and Row 6).
   * Identify the polarity: **Short Leg = Negative (-)**, **Long Leg = Positive (+)**.
   * Connect a jumper wire from the **Short Leg** row to the local **GND (-) rail**.
   * Place your **220Ω resistor** bridging the **Long Leg** row to an empty row nearby (e.g., Row 7).
   * Run a jumper wire from that resistor row straight to ESP32 **GPIO2**.

---

## 💻 Step 2: ESPHome Firmware Configuration

Compile and flash the firmware using your ESPHome server interface. This configuration establishes your local network parameters and maps your MQTT publishing and subscribing pathways.

```yaml
esphome:
  name: esp32

esp32:
  board: nodemcu-32s
  framework:
    type: esp-idf

logger:

# Ensure 'api:' is removed or commented out to prioritize native MQTT broadcast routing
# api:

ota:
  - platform: esphome

wifi:
  ssid: "Billwong-TIME"
  password: "Billwong1234"

  ap:
    ssid: "Esp32 Fallback Hotspot"
    password: "YOUR_HOTSPOT_PASSWORD"

captive_portal:

# --- MQTT BROKER PARAMS ---
mqtt:
  broker: "192.168.X.X" # Students: Input your specific server IP address here

# --- INPUT CHANNELS (Publishing States to MQTT) ---
binary_sensor:
  - platform: gpio
    pin: 27
    name: "Esp32 Lab Motion Sensor"
    device_class: motion

sensor:
  - platform: adc
    pin: 34
    name: "Esp32 Lab Light Level"
    update_interval: 5s
    filters:
      - multiply: 100
    unit_of_measurement: "%"

# --- OUTPUT CHANNELS (Subscribing to Incoming MQTT Commands) ---
switch:
  - platform: gpio
    pin: 2
    name: "ESP32 Onboard LED"
    icon: "mdi:led-on"
