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
- 💾 Adjustable meter offset (for annual calibration)
- 🔄 Reset button for pulse counter
- 💡 Visual LED feedback on each pulse (3 seconds)
- 📦 Modular package-based configuration
- 🌐 Dual language support (English/German)

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
   cp esphome/secrets.yaml.example esphome/secrets.yaml
   ```

   Then edit `esphome/secrets.yaml` with your WiFi credentials:

   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ```

3. Adjust the configuration in `esphome/gas-meter-wemos.yaml`:

   ```yaml
   substitutions:
     devicename: "gas-meter"
     friendly_name: "Gas Meter"
     pulses_per_cubic_meter: "100"  # Adjust for your meter
     initial_meter_offset: "0"       # Set to your current meter reading
   ```

4. **(Optional)** Change localization to German:

   Edit line 50 in `esphome/gas-meter-wemos.yaml`:
   ```yaml
   # Change from:
   translations: !include localization/en.yaml

   # To:
   translations: !include localization/de.yaml
   ```

### 3. Flash the Firmware

**Flash via USB:**

```bash
esphome run esphome/gas-meter-wemos.yaml
```

**Note:** This project uses USB flashing only. OTA updates are not included
for simplicity and security.

## Alternative: Using Remote Packages

Instead of cloning the repository, you can use the **remote package
configuration** that loads all files directly from GitHub. This is perfect
if you don't want to maintain local copies of the configuration files.

### Quick Setup with Remote Packages

1. **Prerequisites:**
   - Install [ESPHome](https://esphome.io/guides/getting_started_command_line.html)
   - Have your WiFi credentials ready

2. **Download only the remote config file:**

   ```bash
   curl -O https://raw.githubusercontent.com/legacycode/ESPHome-Gas-Meter/main/esphome/gas-meter-wemos-remote.yaml
   ```

3. **Create a secrets.yaml in the same directory:**

   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ```

4. **Adjust configuration variables** in `gas-meter-wemos-remote.yaml`:

   ```yaml
   substitutions:
     devicename: "gas-meter-remote"
     friendly_name: "Gas Meter Remote"
     pulses_per_cubic_meter: "100"  # Adjust for your meter
     initial_meter_offset: "0"       # Set to your current meter reading
   ```

5. **Flash via USB:**

   ```bash
   esphome run gas-meter-wemos-remote.yaml
   ```

### How Remote Packages Work

The remote configuration automatically downloads all necessary files from
GitHub on first use:

- **common/packages.yaml** - Core ESPHome components
- **common/boards/esp8266-d1-mini.yaml** - Board configuration
- **gas-meter/packages.yaml** - All gas meter functionality
- **localization/en.yaml** - English translations

Files are cached locally and refreshed every 24 hours (`refresh: 1d`).

### Advantages of Remote Packages

✅ **No repository cloning** - Just one config file needed
✅ **Automatic updates** - Get latest changes from GitHub
✅ **Minimal maintenance** - No local file management
✅ **Same functionality** - 100% feature parity with local version

### Switching to Local Configuration

If you prefer local files for customization, simply:
1. Clone the full repository
2. Use `esphome/gas-meter-wemos.yaml` instead
3. Modify any package files as needed

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

## Architecture

This project uses a **modular package-based structure** for better
maintainability:

```
esphome/
├── common/               # Base configuration
│   ├── boards/          # Board-specific configs
│   ├── core/            # Core ESPHome components
│   └── packages.yaml    # Aggregates all common packages
├── gas-meter/           # Gas meter functionality
│   ├── controls/        # Reset button, offset number
│   ├── core/            # Boot, globals, pulse meter logic
│   ├── sensors/         # Diagnostic sensors
│   ├── led-internal.yaml
│   └── packages.yaml    # Aggregates all gas-meter packages
├── localization/        # Language support (EN/DE)
└── gas-meter-wemos.yaml # Main device configuration
```

The main configuration includes just 4 packages:
- `common/packages.yaml` - Core components (esphome, wifi, api, preferences)
- Board configuration (ESP8266 D1 Mini)
- `gas-meter/packages.yaml` - All gas meter functionality
- `localization/en.yaml` (or de.yaml) - Language translations

## Troubleshooting

### No Pulses Detected

1. Check wiring connections (D2 and GND)
2. Verify the reed contact is positioned correctly near the magnet
3. Test the reed contact with a magnet manually
4. Check the logs: `esphome logs esphome/gas-meter-wemos.yaml`

### Pulses Too Fast/Slow

Adjust the internal filter in `esphome/gas-meter/core/pulse-meter.yaml`:

```yaml
internal_filter: 200ms  # Increase from default 100ms if you get false pulses
```

### Device Not Connecting to WiFi

1. Check `esphome/secrets.yaml` credentials
2. Ensure you're using 2.4 GHz WiFi (ESP8266 doesn't support 5 GHz)
3. Check WiFi signal strength via Home Assistant diagnostic sensors

**Note:** This configuration does not include a fallback WiFi AP. If WiFi
connection fails, you'll need to reflash via USB with corrected credentials.

## What's NOT Included

This is a **simplified configuration** focused on core functionality.
The following features are intentionally excluded:

- ❌ OTA updates (use USB for flashing)
- ❌ MQTT (use Home Assistant API instead)
- ❌ Web server (configure via Home Assistant)
- ❌ Captive portal (no fallback WiFi AP)
- ❌ Time/NTP (not needed for pulse counting)
- ❌ API encryption (suitable for trusted home networks)

**Why simplified?** Faster compilation, smaller firmware, easier to understand,
and fewer dependencies.

**Need these features?** You can add them by creating additional package files
in `esphome/common/core/` and including them in `gas-meter-wemos.yaml`.

## Continuous Integration

This project uses GitHub Actions to automatically build and validate the
firmware on every push and pull request. The workflow:

- ✅ Compiles the YAML configuration
- ✅ Validates the ESPHome configuration syntax
- ✅ Ensures firmware builds successfully

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
