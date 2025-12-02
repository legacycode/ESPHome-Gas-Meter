# ESPHome Gas Meter Reader

[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/blob/main/README.md)
[![de](https://img.shields.io/badge/lang-de-blue.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/blob/main/README_de.md)

ESPHome configuration for reading a gas meter using a reed contact sensor
with a Wemos D1 Mini (ESP8266).

[![Build ESPHome firmware](https://github.com/legacycode/ESPHome-Gas-Meter/actions/workflows/build.yml/badge.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/actions/workflows/build.yml)
![Gas Meter](https://img.shields.io/badge/ESPHome-Compatible-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

- 📊 Real-time gas consumption monitoring
- 🔄 Persistent pulse counter (survives reboots)
- 📈 Accurate meter reading with adjustable offset
- 🏠 Home Assistant Energy Dashboard compatible
- 📡 MQTT support for additional integrations
- 💾 Adjustable meter offset (for annual calibration)
- 🔄 Reset button for pulse counter
- 💡 Visual LED feedback on each pulse
- 🌐 Web interface for configuration

## Hardware Requirements

- **Wemos D1 Mini** (or any ESP8266 board)
- **Reed contact sensor** (normally open)
- Gas meter with magnetic pulse output
- USB cable for power and programming

## Wiring

Connect the reed contact sensor to your Wemos D1 Mini:

```text
Reed Contact Sensor
┌─────────────┐
│    Reed     │
│   Contact   │
└──┬──────┬───┘
   │      │
   │      └──────────> GND (Wemos D1 Mini)
   │
   └─────────────────> D2 (GPIO4, Wemos D1 Mini)
```

### Pin Configuration

| Component    | Wemos Pin | GPIO  | Description                          |
|--------------|-----------|-------|--------------------------------------|
| Reed Contact | D2        | GPIO4 | Pulse input from gas meter           |
| Internal LED | D4        | GPIO2 | Status indicator (flashes on pulse)  |

**Note:** The internal pull-up resistor is enabled in the configuration,
so no external resistor is needed.

## Installation

### 1. Prerequisites

- Install [ESPHome](https://esphome.io/guides/getting_started_command_line.html)
- Have your WiFi credentials ready
- Know your gas meter's pulses per cubic meter (check meter specifications)

### 2. Configuration

1. Clone this repository:

   ```bash
   git clone https://github.com/legacycode/ESPHome-Gas-Meter.git
   cd ESPHome-Gas-Meter
   ```

2. Create a `secrets.yaml` file from the template:

   ```bash
   cp secrets.yaml.example secrets.yaml
   ```

   Then edit `secrets.yaml` with your credentials:

   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ap_password: "FallbackAP"
   encryption_key: "your-32-char-encryption-key"
   ota_password: "YourOTAPassword"
   ```

   Generate secure keys using:
   ```bash
   # For encryption_key (32 bytes base64)
   openssl rand -base64 32

   # For ota_password (hex string)
   openssl rand -hex 16
   ```

3. Adjust the configuration in `gas-meter-wemos-en.yaml` or
   `gas-meter-wemos-de.yaml`:

   ```yaml
   substitutions:
     pulses_per_cubic_meter: "100"  # Adjust for your meter
     initial_meter_offset: "0"       # Set to your current meter reading
   ```

### 3. Flash the Firmware

**First time (via USB):**

```bash
esphome run gas-meter-wemos-en.yaml
```

**Over-the-air updates (after initial flash):**

```bash
esphome run gas-meter-wemos-en.yaml --device gas-meter.local
```

## Configuration Options

### Pulses Per Cubic Meter

Common values for gas meters:

- **1 pulse/m³**: 1 pulse = 1 cubic meter
- **10 pulses/m³**: 1 pulse = 0.1 cubic meters
- **100 pulses/m³**: 1 pulse = 0.01 cubic meters (most common)
- **1000 pulses/m³**: 1 pulse = 0.001 cubic meters

Check your gas meter specification or the meter itself for the correct value.

### Initial Meter Offset

Set this to your current gas meter reading to start tracking from the
actual meter value:

```yaml
initial_meter_offset: "1234"  # Your current meter reading in m³
```

You can also adjust this later via Home Assistant using the
**"Meter Offset"** entity.

## Home Assistant Integration

### Available Entities

After adding the device to Home Assistant, you'll have access to:

| Entity             | Type          | Description                            |
|--------------------|---------------|----------------------------------------|
| **Flow Rate**      | Sensor        | Current gas flow rate (m³/h)           |
| **Total**          | Sensor        | Measured consumption since reset (m³)  |
| **Meter Reading**  | Sensor        | Actual meter reading (Offset + Total)  |
| **Total Pulses**   | Sensor        | Raw pulse count                        |
| **Meter Offset**   | Number        | Adjustable offset for calibration      |
| **Reset Pulses**   | Button        | Reset the pulse counter to zero        |
| **Status**         | Binary Sensor | Device online/offline status           |
| **LED**            | Light         | Control the status LED                 |
| **WiFi Signal**    | Sensor        | WiFi signal strength                   |
| **Uptime**         | Sensor        | Device uptime                          |

### Energy Dashboard Integration

1. Go to **Settings** → **Dashboards** → **Energy**
2. Under **Gas consumption**: **Add Gas Source**
3. Select: **sensor.gas_meter_meter_reading**
4. Confirm

The Energy Dashboard will automatically track your daily, monthly, and yearly
gas consumption.

### Annual Calibration

To calibrate with your energy provider's reading:

1. Compare **Meter Reading** sensor with your physical meter
2. Calculate the difference: `actual_reading - sensor_reading`
3. Adjust **Meter Offset** by adding the difference
4. The **Meter Reading** now matches your physical meter

**Example:**

- Physical meter: 1456 m³
- Sensor reading: 1450 m³
- Difference: +6 m³
- Current offset: 1234 m³
- New offset: 1234 + 6 = **1240 m³**

## MQTT Integration

The device publishes to MQTT topics under `esphome/gas-meter/`.

Disable Home Assistant MQTT Discovery if you're using the API:

```yaml
mqtt:
  discovery: false  # Already enabled in config
```

## Troubleshooting

### No Pulses Detected

1. Check wiring connections (D2 and GND)
2. Verify the reed contact is positioned correctly near the magnet
3. Test the reed contact with a magnet manually
4. Check the logs: `esphome logs gas-meter-wemos-en.yaml`

### Pulses Too Fast/Slow

Adjust the internal filter to prevent false triggers:

```yaml
internal_filter: 100ms  # Increase if you get false pulses
```

### Device Not Connecting to WiFi

1. Check `secrets.yaml` credentials
2. Use the fallback AP: Connect to "Gas Meter Fallback"
3. Configure WiFi through the captive portal

## Files

- `gas-meter-wemos-en.yaml` - English configuration
- `gas-meter-wemos-de.yaml` - German configuration (uses "ae", "oe", "ue"
  for MQTT compatibility)
- `secrets.yaml.example` - Template for WiFi and API credentials
- `secrets.yaml` - Your WiFi and API credentials (not included in git,
  create from template)
- `.github/workflows/build.yml` - GitHub Actions workflow for automated
  builds

## Continuous Integration

This project uses GitHub Actions to automatically build and validate the
firmware on every push and pull request. The workflow:

- ✅ Compiles both English and German YAML configurations
- ✅ Validates the ESPHome configuration syntax
- ✅ Ensures firmware builds successfully

The workflow runs on the `main` branch and uses the `secrets.yaml.example`
template for build validation.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE)
file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgments

- Built with [ESPHome](https://esphome.io/)
- Compatible with [Home Assistant](https://www.home-assistant.io/)

## Support

If you encounter any issues or have questions, please open an issue on GitHub.

---

**Disclaimer:** This project involves electrical components and gas meter
modifications. Ensure you comply with local regulations and safety standards.
The authors are not responsible for any damage or injury resulting from the
use of this project.
