# ESPHome Gas Meter Reader - Project Context

## Project Overview

This project provides an ESPHome configuration for reading gas meters using a
reed contact sensor with a Wemos D1 Mini (ESP8266). It enables real-time
monitoring of gas consumption and integrates seamlessly with Home Assistant's
Energy Dashboard.

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
esp-home-gas-meter/
├── .github/
│   └── workflows/
│       └── build.yml              # GitHub Actions CI/CD
├── .esphome/                      # Build artifacts (gitignored)
├── .claude/                       # Claude Code config (gitignored)
├── .devcontainer/                 # Dev container config (gitignored)
├── gas-meter-wemos-en.yaml        # English ESPHome config
├── gas-meter-wemos-de.yaml        # German ESPHome config
├── secrets.yaml                   # WiFi/API credentials (gitignored)
├── secrets.yaml.example           # Template for secrets
├── .gitignore                     # Git ignore rules
├── .markdownlint.json             # Markdown linting config
├── README.md                      # English documentation
├── README_de.md                   # German documentation
├── LICENSE                        # MIT License
└── claude.md                      # This file - project context
```

## Configuration Files

### ESPHome YAML Structure

Both `gas-meter-wemos-en.yaml` and `gas-meter-wemos-de.yaml` have the same
structure:

1. **Substitutions**: Variables for customization
   - `devicename`: Internal device name
   - `friendly_name`: Display name in Home Assistant
   - `device_comment`: Description
   - `pulses_per_cubic_meter`: Meter calibration (default: 100)
   - `initial_meter_offset`: Starting meter reading

2. **Core Components**:
   - ESPHome core configuration
   - ESP8266 board definition
   - Logger, API, MQTT, OTA, WiFi

3. **Globals**:
   - `total_pulses_global`: Persistent pulse counter
   - `meter_offset`: Persistent meter offset (in m³)

4. **Sensors**:
   - **Flow Rate**: Current gas flow (m³/h)
   - **Total**: Measured consumption (m³)
   - **Meter Reading**: Total with offset (m³)
   - **Total Pulses**: Raw pulse count
   - **WiFi Signal**: Signal strength
   - **Uptime**: Device uptime

5. **Controls**:
   - **Meter Offset** (Number): Adjustable offset
   - **Reset Pulses** (Button): Reset pulse counter
   - **LED** (Light): Status indicator

### Key Features

- **Persistent Storage**: Pulse count and offset survive reboots
- **LED Feedback**: Flashes for 3 seconds on each pulse
- **Debouncing**: 100ms internal filter prevents false triggers
- **Timeout**: 2-minute timeout sets flow to zero if no pulses
- **Dual Protocol**: Both Home Assistant API and MQTT support

## MQTT vs. German Naming

The German configuration (`gas-meter-wemos-de.yaml`) uses "ae", "oe", "ue"
instead of umlauts (ä, ö, ü) for MQTT compatibility:

- `Gaszaehler` instead of `Gaszähler`
- `Zaehlerstand` instead of `Zählerstand`
- `zuruecksetzen` instead of `zurücksetzen`

This ensures proper display in MQTT topics and avoids UTF-8 encoding issues.

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
3. Adjust "Meter Offset" by adding the difference
4. The "Meter Reading" now matches the physical meter

Example:

- Physical meter: 1456 m³
- Sensor reading: 1450 m³
- Difference: +6 m³
- Current offset: 1234 m³
- New offset: 1234 + 6 = **1240 m³**

## GitHub Actions CI/CD

The workflow (`.github/workflows/build.yml`) automatically:

1. Builds both English and German configurations
2. Validates ESPHome syntax
3. Creates firmware artifacts (.bin and .elf files)
4. Uploads artifacts (30-day retention)

Triggers:

- Push to main/master branch
- Pull requests
- Manual workflow dispatch

## Development Notes

### Markdownlint Configuration

`.markdownlint.json` allows:

- Line length up to 120 characters (excludes tables and code blocks)
- Inline HTML for language switcher (`<div>`, `<a>`)

### Secrets Management

`secrets.yaml` is gitignored and contains:

- `wifi_ssid`: WiFi network name
- `wifi_password`: WiFi password
- `ap_password`: Fallback AP password
- `encryption_key`: 32-character API encryption key
- `ota_password`: OTA update password

Use `secrets.yaml.example` as a template.

## Common Meter Configurations

### Pulses Per Cubic Meter

- **1 pulse/m³**: 1 pulse = 1 cubic meter
- **10 pulses/m³**: 1 pulse = 0.1 cubic meters
- **100 pulses/m³**: 1 pulse = 0.01 cubic meters (most common)
- **1000 pulses/m³**: 1 pulse = 0.001 cubic meters

Check your gas meter specification plate for the correct value.

## Troubleshooting

### No Pulses Detected

1. Verify wiring (D2 and GND)
2. Check reed contact position near magnet
3. Test reed contact manually with a magnet
4. Check logs: `esphome logs gas-meter-wemos-en.yaml`

### False Pulses

Increase debounce filter:

```yaml
internal_filter: 200ms  # Increase from default 100ms
```

### WiFi Connection Issues

1. Verify `secrets.yaml` credentials
2. Use fallback AP: "Gas Meter Fallback" / "Gaszaehler Fallback"
3. Configure through captive portal

## Build and Flash

### First Flash (USB)

```bash
esphome run gas-meter-wemos-en.yaml
```

### OTA Updates

```bash
esphome run gas-meter-wemos-en.yaml --device gas-meter.local
```

## License

MIT License - See LICENSE file

## Project History

### Initial Setup

1. Created ESPHome configuration for Wemos D1 Mini
2. Configured reed contact on D2 (GPIO4)
3. Implemented persistent pulse counter
4. Added adjustable meter offset

### German Localization

1. Created German version with MQTT-compatible names
2. Translated all sensor and entity names
3. Used "ae", "oe", "ue" for umlauts

### Documentation

1. Created bilingual README (English/German)
2. Added language switcher in both READMEs
3. Fixed all markdownlint issues
4. Created comprehensive project context

### CI/CD

1. Added GitHub Actions workflow
2. Automated build validation
3. Artifact creation and upload

## Pin Change History

**Original**: D3 (GPIO0)
**Updated**: D2 (GPIO4)

The pin was changed per user request. Both YAML files and documentation were
updated to reflect this change.

## Important Implementation Details

### Pulse Counting

The configuration uses `internal_filter_mode: PULSE` which counts complete
pulse cycles (close + open = 1 pulse). Using `EDGE` would incorrectly count
both close and open as separate pulses.

### LED Feedback

The internal LED (GPIO2) provides visual confirmation:

- Turns ON when pulse detected
- Stays ON for 3 seconds
- Turns OFF automatically

### Global Variables

Two global variables ensure persistence:

1. `total_pulses_global`: Integer pulse count
2. `meter_offset`: Float meter offset in m³

Both have `restore_value: true` to survive reboots.

### Reset Functionality

The "Reset Pulses" button:

1. Sets `total_pulses_global` to 0
2. Calls `gas_pulse_meter.set_total_pulses(0)`
3. Does NOT reset the meter offset

This allows starting fresh measurements without losing calibration.

## Multi-Language Support

The repository supports both English and German:

- **README.md**: English (default)
- **README_de.md**: German
- Both have language switcher at the top
- GitHub displays English README by default

## Build Status

Build status is shown via GitHub Actions badge in both READMEs:

```markdown
![Build Status](https://github.com/legacycode/ESPHome-Gas-Meter/workflows/Build%20ESPHome%20firmware/badge.svg)
```

## Future Considerations

### Potential Enhancements

1. **Battery Backup**: Add RTC with battery for power-off resilience
2. **Display**: Add small OLED for local reading
3. **Multiple Meters**: Support for water/electricity meters
4. **Advanced Analytics**: Local trend calculation
5. **Notifications**: Alert on unusual consumption patterns

### Known Limitations

1. Requires stable WiFi connection
2. Depends on Home Assistant for long-term storage
3. No local display without modification
4. Manual calibration required annually

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

## Repository URLs

The project is hosted at:

- **HTTPS**: https://github.com/legacycode/ESPHome-Gas-Meter
- **Git Clone**: https://github.com/legacycode/ESPHome-Gas-Meter.git

All URLs have been updated in:
- README.md (language badges, build badge, clone instructions)
- README_de.md (language badges, build badge, clone instructions)

## Contact and Support

For issues or questions:

- Open an issue on GitHub
- Check existing issues first
- Provide logs and configuration when reporting problems

---

**Last Updated**: 2024-12-02
**Project Status**: Active
**Maintenance**: Community-maintained
