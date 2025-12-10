# ESPHome Gas Meter Reader - Project Context

## Project Overview

This project provides an ESPHome configuration for reading gas meters using a
reed contact sensor with a Wemos D1 Mini (ESP8266). It enables real-time
monitoring of gas consumption and integrates seamlessly with Home Assistant's
Energy Dashboard.

The project uses a **modular package-based structure** for better maintainability
and reusability.

## Hardware Setup

### Components

- **Wemos D1 Mini** (ESP8266)
- **Reed contact sensor** (normally open)
- Gas meter with magnetic pulse output

### Wiring Configuration

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

**Pin Mapping:**

- **D2 (GPIO4)**: Reed contact input
- **D4 (GPIO2)**: Internal LED for visual feedback

## File Structure

```text
esphome/
├── common/
│   ├── boards/
│   │   └── esp8266-d1-mini.yaml       # Board config with pins & logger
│   ├── core/
│   │   ├── api.yaml                    # Home Assistant API (no encryption)
│   │   ├── esphome.yaml                # Core ESPHome settings
│   │   ├── preferences.yaml            # Flash storage settings
│   │   └── wifi.yaml                   # WiFi (no fallback AP)
│   └── packages.yaml                   # ⭐ Aggregates all common packages
├── gas-meter/
│   ├── controls/
│   │   ├── offset-number.yaml         # Adjustable meter offset
│   │   └── reset-button.yaml          # Reset pulse counter
│   ├── core/
│   │   ├── globals.yaml                # Global variables
│   │   ├── meter-reading.yaml         # Meter reading with offset
│   │   └── pulse-meter.yaml           # Pulse meter sensor
│   ├── led-internal.yaml               # Internal LED for pulse feedback
│   └── packages.yaml                   # ⭐ Aggregates all gas-meter packages
├── localization/
│   ├── de/
│   │   └── translations.yaml           # German translations
│   ├── en/
│   │   └── translations.yaml           # English translations
│   ├── de.yaml                         # German wrapper
│   └── en.yaml                         # English wrapper
├── gas-meter-wemos.yaml                # Main device configuration
├── secrets.yaml                        # WiFi credentials (gitignored)
└── secrets.yaml.example                # Template for secrets
```

## Configuration Files

### Main Configuration: gas-meter-wemos.yaml

This is the main device configuration that uses aggregated packages for maximum simplicity:

```yaml
packages:
  # Base Configuration (core, board, wifi, api)
  common:            !include common/packages.yaml

  # Gas Meter Functionality (sensors, controls, LED)
  gas_meter:         !include gas-meter/packages.yaml

  # Localization (change to "de.yaml" for German)
  translations:      !include localization/en.yaml
```

**Included Features:**
- ✅ Full gas meter functionality (LED, button, offset, all sensors)
- ✅ WiFi connection (no fallback AP)
- ✅ Home Assistant API (no encryption)
- ✅ Localization support (English/German)
- ❌ OTA updates (use USB for flashing)
- ❌ MQTT
- ❌ Web server
- ❌ Captive portal
- ❌ Time/NTP

**Key Substitutions:**
- `devicename`: Internal device name
- `friendly_name`: Display name in Home Assistant
- `pulses_per_cubic_meter`: Meter calibration (default: 100)
- `initial_meter_offset`: Starting meter reading

### Remote Configuration: gas-meter-wemos-remote.yaml

This is an alternative configuration that loads all packages directly from GitHub
instead of requiring local files. Perfect for users who don't want to clone the
repository.

```yaml
substitutions:
  devicename: "gas-meter-remote"
  friendly_name: "Gas Meter Remote"
  device_comment: "Gas Meter Reader with Reed Contact"
  pulses_per_cubic_meter: "100"
  initial_meter_offset: "0"

packages:
  gas_meter:
    url: https://github.com/legacycode/ESPHome-Gas-Meter
    files: [
      esphome/common/packages.yaml,
      esphome/common/boards/esp8266-d1-mini.yaml,
      esphome/gas-meter/packages.yaml,
      esphome/localization/en.yaml
    ]
    ref: main
    refresh: 1d
```

**How It Works:**
- Downloads all packages from GitHub on first use
- Caches files locally for 24 hours (`refresh: 1d`)
- Requires only one config file + secrets.yaml
- Automatically gets updates from the repository
- 100% feature parity with local version

**Key Requirements:**
- All substitutions must be defined BEFORE the packages section
- Uses `files:` array to load multiple files from the same repository
- Files are referenced relative to repository root
- WiFi credentials are loaded from `secrets.yaml` (wifi_ssid, wifi_password)
- No wifi_ssid_sub/wifi_password_sub substitutions needed

**Usage:**
1. Download `gas-meter-wemos-remote.yaml` from repository
2. Create `secrets.yaml` with WiFi credentials
3. Adjust substitutions as needed
4. Flash: `esphome run gas-meter-wemos-remote.yaml`

**Advantages:**
- ✅ No repository cloning needed
- ✅ Automatic updates from GitHub
- ✅ Minimal maintenance
- ✅ Perfect for quick testing or deployment

### Package Structure

The configuration uses a **two-level package hierarchy** for maximum simplicity:

**Level 1 - Aggregated Packages** (included in `gas-meter-wemos.yaml`):

1. **`common/packages.yaml`** - All base components:
   - Core: `esphome.yaml`, `preferences.yaml`, `wifi.yaml`, `api.yaml`
   - Board: `esp8266-d1-mini.yaml`

2. **`gas-meter/packages.yaml`** - All gas meter components:
   - LED: `led-internal.yaml`
   - Core: `globals.yaml`, `pulse-meter.yaml`, `meter-reading.yaml`
   - Controls: `reset-button.yaml`, `offset-number.yaml`

3. **`localization/en.yaml` or `de.yaml`** - Language translations

**Level 2 - Individual Component Packages** (included in aggregated packages):

See the `common/packages.yaml` and `gas-meter/packages.yaml` files for the complete
list of individual components. This two-level approach keeps the main configuration
extremely clean while maintaining full modularity

### Globals

Two global variables ensure persistence:

- `total_pulses_global`: Integer pulse count (survives reboots)
- `meter_offset`: Float meter offset in m³ (survives reboots)

### Sensors

**Sensors:**
- **Flow Rate** (`pulse_meter`): Current gas flow in m³/h
- **Total** (`pulse_meter.total`): Total consumption in m³
- **Meter Reading** (`template`): Total + Offset for Energy Dashboard
- **Total Pulses**: Raw pulse count

### Controls

**Button:**
- **Reset Pulses**: Resets pulse counter to zero

**Number:**
- **Meter Offset**: Adjustable offset for calibration (0-999999 m³)

### LED Feedback

The internal LED provides visual confirmation:
- Turns ON when pulse detected
- Stays ON for 3 seconds
- Turns OFF automatically

## Key Features

- **Persistent Storage**: Pulse count and offset survive reboots
- **LED Feedback**: Flashes for 3 seconds on each pulse
- **Debouncing**: 100ms internal filter prevents false triggers
- **Timeout**: 2-minute timeout sets flow to zero if no pulses
- **Modular Design**: Easy to add/remove features via packages
- **Dual Language Support**: English and German localizations

## Localization

The project supports both English and German:

**To change language**, edit line 47 in `gas-meter-wemos.yaml`:

```yaml
# English (default)
translations: !include localization/en.yaml

# German
translations: !include localization/de.yaml
```

**German naming** uses "ae", "oe", "ue" instead of umlauts for MQTT compatibility.

## Home Assistant Integration

### Energy Dashboard

The main sensor for the Energy Dashboard is:

- **English**: `sensor.gas_meter_meter_reading`
- **German**: `sensor.gaszaehler_zaehlerstand`

This sensor has:
- `state_class: total_increasing`
- `device_class: gas`
- Includes the meter offset for accurate readings

### Annual Calibration Workflow

1. Compare Home Assistant reading with physical meter
2. Calculate difference: `physical - sensor`
3. Adjust "Meter Offset" in Home Assistant by adding the difference
4. The "Meter Reading" now matches the physical meter

Example:
- Physical meter: 1456 m³
- Sensor reading: 1450 m³
- Difference: +6 m³
- Current offset: 1234 m³
- New offset: **1240 m³**

## Secrets Management

`secrets.yaml` contains only WiFi credentials:

```yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
```

Use `secrets.yaml.example` as a template.

## Common Meter Configurations

### Pulses Per Cubic Meter

- **1 pulse/m³**: 1 pulse = 1 cubic meter
- **10 pulses/m³**: 1 pulse = 0.1 cubic meters
- **100 pulses/m³**: 1 pulse = 0.01 cubic meters (most common)
- **1000 pulses/m³**: 1 pulse = 0.001 cubic meters

Check your gas meter specification plate for the correct value.

## Build and Flash

### First Flash (USB)

```bash
esphome run esphome/gas-meter-wemos.yaml
```

### Validation Only

```bash
esphome config esphome/gas-meter-wemos.yaml
```

### Compilation Only

```bash
esphome compile esphome/gas-meter-wemos.yaml
```

## GitHub Actions CI/CD

The workflow (`.github/workflows/build.yml`) automatically:

1. Builds the configuration
2. Validates ESPHome syntax
3. Runs on push to main branch or pull requests

## Troubleshooting

### No Pulses Detected

1. Verify wiring (D2 and GND)
2. Check reed contact position near magnet
3. Test reed contact manually with a magnet
4. Check logs: `esphome logs esphome/gas-meter-wemos.yaml`

### False Pulses

Increase debounce filter in `gas-meter/core/pulse-meter.yaml`:

```yaml
internal_filter: 200ms  # Increase from default 100ms
```

### WiFi Connection Issues

1. Verify `secrets.yaml` credentials
2. Check WiFi signal strength (diagnostic sensor)
3. Ensure 2.4 GHz WiFi (ESP8266 doesn't support 5 GHz)

## Simplified Architecture

This project intentionally excludes certain features for simplicity:

**Not Included:**
- ❌ OTA updates - Flash via USB for security
- ❌ MQTT - Use Home Assistant API instead
- ❌ Web server - Configure via Home Assistant
- ❌ Captive portal - No WiFi fallback AP
- ❌ Time/NTP - Not needed for pulse counting
- ❌ API encryption - Suitable for trusted networks

**Why Simplified?**
- Faster compilation
- Smaller firmware size
- Easier to understand for beginners
- Less attack surface
- Fewer dependencies

**Adding Features:**
If you need OTA, MQTT, or other features, create additional package files
in `common/core/` and include them in `gas-meter-wemos.yaml`.

## License

MIT License - See LICENSE file

## Project History

### 2024-12-05: Remote Packages Support
- Added `gas-meter-wemos-remote.yaml` for loading packages from GitHub
- Updated README.md and README_de.md with Remote Packages documentation
- Changed remote package ref from feature branch to `main`
- Enables usage without cloning repository

### 2024-12-04: Simplified Architecture
- Removed OTA, MQTT, web server, captive portal, time/NTP
- Consolidated to single configuration file
- Simplified secrets (WiFi only)
- Removed API encryption requirement
- Moved connectivity packages (api, wifi) from connectivity/ to core/
- Moved LED component from common/components/ to gas-meter/
- Simplified directory structure (removed connectivity and components folders)
- **Added aggregated package files** (`common/packages.yaml`, `gas-meter/packages.yaml`)
- Main config now includes only 4 packages instead of 10+ individual files
- Moved board configuration from common/packages.yaml to main device config
- Split boot logic from esphome.yaml into gas-meter/core/boot.yaml

### 2024-12-02: Package-Based Structure
- Migrated from monolithic to modular package structure
- Created granular packages for each component
- Implemented localization via wrapper files
- Added remote package support (later removed)

### Initial Setup
- Created ESPHome configuration for Wemos D1 Mini
- Configured reed contact on D2 (GPIO4)
- Implemented persistent pulse counter
- Added adjustable meter offset

## Pin Change History

**Original**: D3 (GPIO0)
**Updated**: D2 (GPIO4)

The pin was changed per user request. All configurations and documentation
reflect this change.

## Important Implementation Details

### Pulse Counting

The configuration uses `internal_filter_mode: PULSE` which counts complete
pulse cycles (close + open = 1 pulse). Using `EDGE` would incorrectly count
both close and open as separate pulses.

### Reset Functionality

The "Reset Pulses" button:
1. Sets `total_pulses_global` to 0
2. Calls `gas_pulse_meter.set_total_pulses(0)`
3. Does NOT reset the meter offset

This allows starting fresh measurements without losing calibration.

## Safety and Compliance

**Important**: This project involves:
- Electrical components (low voltage, but still requires care)
- Gas meter modifications (check local regulations)
- Potential warranty implications

Always:
- Comply with local regulations
- Consult with your energy provider if required
- Test thoroughly before relying on readings
- Keep manual meter readings for backup

---

**Last Updated**: 2024-12-04
**Project Status**: Active
**Maintenance**: Community-maintained
