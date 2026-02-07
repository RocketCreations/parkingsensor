# Parking Sensor

An Arduino/ESP-based parking sensor that uses an ultrasonic sensor and Adafruit Neopixel LED ring to provide visual distance feedback when parking your car.

## How Do These Agents Work?

In the context of this project, **"agents"** refer to **MySensors sensor nodes**. Each parking sensor device acts as an independent agent (node) in a MySensors network.

### Key Points About Agents:

**Do I need to specify a project for each agent interaction?**  
**No!** Each parking sensor agent is self-contained and does NOT require per-agent project configuration. The agent:
- Works **standalone** by default (no network required)
- Can optionally connect to **one central controller** (like Home Assistant, Domoticz, etc.)
- Is configured at **compile-time** through the Arduino sketch settings
- Requires **no runtime project assignment** - just upload the code and it works

### How Agents Work

Each parking sensor agent operates in one of two modes:

#### 1. Standalone Mode (Default)
- The sensor measures distance and displays visual feedback via the LED ring
- No wireless communication
- Perfect for simple garage parking assistance
- Just power it up and it works!

#### 2. Network Mode (Optional)
- Uncomment `#define SEND_STATUS_TO_CONTROLLER` in the code
- Agent reports parking status to your home automation controller
- Uses MySensors wireless protocol (NRF24 or RFM69 radio)
- Participates in a self-healing mesh network

## Features

- **Visual Distance Feedback**: LED ring changes from green → yellow → red as you get closer
- **Blinking Alert**: Red blinking when too close (panic mode)
- **Auto-Off**: LEDs turn off after parking timeout to save power
- **Wireless Reporting**: Optional MySensors integration for home automation
- **Standalone Operation**: Works without any controller or network

## Hardware Requirements

### Required Components
- Arduino Uno, Nano, or ESP8266/ESP32
- HC-SR04 Ultrasonic Distance Sensor
- Adafruit Neopixel Ring/Strip (24 LEDs recommended)
- Power supply (5V, sufficient for LEDs - see power requirements below)

### Optional Components (for Network Mode)
- NRF24L01+ Radio Module OR
- RFM69 Radio Module

### Pin Connections

| Component | Arduino Pin | Notes |
|-----------|-------------|-------|
| Neopixel Data | Digital Pin 6 | NEO_PIN |
| HC-SR04 Trigger | Digital Pin 3 | TRIGGER_PIN |
| HC-SR04 Echo | Digital Pin 2 | ECHO_PIN |
| NRF24/RFM69 | SPI Pins | Only if using network mode |

**⚠️ Important**: Feed the Neopixel LEDs and ultrasonic sensor from a **separate power supply**, not directly from Arduino. The Arduino cannot provide enough current for 24 LEDs.

## LED Color Guide

The LED ring provides intuitive visual feedback:

| Distance Range | LED Color | Meaning |
|---------------|-----------|---------|
| > 100cm | No LEDs | Object too far / no object detected |
| 100cm - 60cm | Green → Yellow gradient | Safe to continue |
| 60cm - 50cm | Yellow → Orange | Slow down, getting close |
| < 50cm | Red (solid) | STOP! Too close |
| At minimum | Red (blinking) | DANGER! Panic mode |

More LEDs light up as you get closer. At the panic distance, all LEDs blink red rapidly.

## Configuration

All configuration is done by modifying constants in `ParkingSensor.ino`:

### Distance Settings
```cpp
#define MAX_DISTANCE 100      // Maximum detection range (cm)
#define PANIC_DISTANCE 50     // Red warning threshold (cm)
#define PARKED_DISTANCE 60    // "Parked" status threshold (cm)
```

### LED Settings
```cpp
#define NUMPIXELS 24          // Number of LEDs in your ring/strip
#define MAX_INTESITY 20       // LED brightness (percentage, 0-100)
```

### Network Settings (Network Mode Only)
```cpp
#define SEND_STATUS_TO_CONTROLLER  // Uncomment to enable
#define MY_RADIO_NRF24            // OR
#define MY_RADIO_RFM69            // Choose your radio type
```

## Installation & Setup

### 1. Install Required Libraries
```bash
# In Arduino IDE, install via Library Manager:
- Adafruit NeoPixel
- NewPing
- MySensors (only if using network mode)
```

### 2. Configure the Code
1. Open `ParkingSensor.ino` in Arduino IDE
2. Adjust distance thresholds for your garage (optional)
3. Set LED brightness (start with 20% to reduce power consumption)
4. For network mode: uncomment `SEND_STATUS_TO_CONTROLLER` and select radio type

### 3. Upload to Arduino
1. Connect your Arduino via USB
2. Select the correct board and port in Arduino IDE
3. Click Upload

### 4. Hardware Assembly
1. Connect the ultrasonic sensor to pins 2 and 3
2. Connect Neopixel data line to pin 6
3. Provide separate 5V power to the Neopixels
4. Mount the sensor at appropriate height in your garage

### 5. Calibration
1. Power on and position your car at the ideal "parked" position
2. Note the distance reported in Serial Monitor (9600 baud)
3. Adjust `PARKED_DISTANCE` if needed

## MySensors Network Integration

### What is MySensors?
MySensors is a home automation framework that enables:
- **Wireless sensor networks** with self-healing mesh topology
- **Gateway** to connect to controllers (Home Assistant, Domoticz, OpenHAB)
- **Multiple sensor nodes** (agents) reporting to one controller

### Setting Up MySensors Mode

1. **Install MySensors Library**
2. **Choose Radio Module**: Uncomment either `MY_RADIO_NRF24` or `MY_RADIO_RFM69`
3. **Enable Controller Communication**: Uncomment `#define SEND_STATUS_TO_CONTROLLER`
4. **Configure Gateway**: Set up a MySensors gateway connected to your controller
5. **Upload & Deploy**: The sensor will automatically join the network

### What Gets Reported?
When in network mode, the agent sends:
- **Parking Status**: Binary sensor (parked/not parked)
- **Update Interval**: Maximum every 5 seconds
- **Sensor Type**: Reported as a "DOOR" sensor in MySensors

## Power Consumption

LED power consumption varies by brightness and number lit:
- At 20% intensity: ~50-100mA per LED
- 24 LEDs all lit at 20%: ~1.2-2.4A
- Recommended: 5V 3A power supply for safety margin

To reduce power:
- Lower `MAX_INTESITY` (default 20%)
- Use fewer LEDs (adjust `NUMPIXELS`)
- Auto-off feature turns LEDs off after parking timeout

## Troubleshooting

### LEDs Don't Light Up
- Check power supply to Neopixels (separate from Arduino)
- Verify data pin connection (pin 6)
- Test with simple Neopixel example sketch

### Sensor Reads Zero
- Check wiring to HC-SR04 (trigger pin 3, echo pin 2)
- Ensure sensor has 5V power
- Code filters out zero readings (up to 10 consecutive)

### Erratic Distance Readings
- Ensure target surface is relatively flat
- Avoid very acute angles
- Check for ultrasonic interference from other devices

### Network Mode Not Working
- Verify radio module connections (SPI pins)
- Check radio type matches code configuration
- Ensure MySensors gateway is powered and accessible

## Customization Ideas

- **Different LED Patterns**: Modify the color calculation in the loop
- **Sound Alert**: Add a buzzer for audio feedback
- **Multiple Sensors**: Deploy multiple parking sensor agents in different garage spots
- **MQTT Integration**: Use MySensors MQTT gateway for custom integrations
- **Temperature Compensation**: Adjust ultrasonic readings based on temperature

## License

This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License version 2 as published by the Free Software Foundation.

Based on MySensors example by Henrik Ekblad  
MySensors Documentation: http://www.mysensors.org

## Support

- MySensors Forum: http://forum.mysensors.org
- MySensors Documentation: http://www.mysensors.org
- Arduino Library Documentation: https://www.arduino.cc/reference/en/
