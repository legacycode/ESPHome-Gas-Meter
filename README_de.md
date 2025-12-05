# ESPHome Gaszähler-Ausleser

[![en](https://img.shields.io/badge/lang-en-red.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/blob/main/README.md)
[![de](https://img.shields.io/badge/lang-de-blue.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/blob/main/README_de.md)

ESPHome-Konfiguration zum Auslesen eines Gaszählers mit einem
Reed-Kontakt-Sensor und einem Wemos D1 Mini (ESP8266).

[![Build ESPHome firmware](https://github.com/legacycode/ESPHome-Gas-Meter/actions/workflows/build.yml/badge.svg)](https://github.com/legacycode/ESPHome-Gas-Meter/actions/workflows/build.yml)
![Gas Meter](https://img.shields.io/badge/ESPHome-Kompatibel-blue)
![License](https://img.shields.io/badge/Lizenz-MIT-green)

## Features

- 📊 Echtzeit-Gasverbrauchsüberwachung
- 🔄 Persistenter Impulszähler (überlebt Neustarts)
- 📈 Präzise Zählerstandserfassung mit einstellbarem Offset
- 🏠 Kompatibel mit Home Assistant Energie-Dashboard
- 💾 Einstellbarer Zählerstand-Offset (für jährliche Kalibrierung)
- 🔄 Reset-Button für Impulszähler
- 💡 Visuelles LED-Feedback bei jedem Impuls (3 Sekunden)
- 📦 Modulare paketbasierte Konfiguration
- 🌐 Zweisprachige Unterstützung (Deutsch/Englisch)

## Hardware-Anforderungen

- **Wemos D1 Mini** (oder ein beliebiges ESP8266-Board)
- **Reed-Kontakt-Sensor** (Öffner-Kontakt)
- Gaszähler mit magnetischem Impulsausgang
- USB-Kabel für Stromversorgung und Programmierung

## Verkabelung

Verbinden Sie den Reed-Kontakt-Sensor mit Ihrem Wemos D1 Mini:

```text
Reed-Kontakt-Sensor
┌─────────────┐
│    Reed     │
│   Kontakt   │
└──┬──────┬───┘
   │      │
   │      └──────────> GND (Wemos D1 Mini)
   │
   └─────────────────> D2 (GPIO4, Wemos D1 Mini)
```

### Pin-Konfiguration

| Komponente   | Wemos Pin | GPIO  | Beschreibung                     |
|--------------|-----------|-------|----------------------------------|
| Reed-Kontakt | D2        | GPIO4 | Impulseingang vom Gaszähler      |
| Interne LED  | D4        | GPIO2 | Statusanzeige (blinkt bei Impuls)|

**Hinweis:** Der interne Pull-up-Widerstand ist in der Konfiguration
aktiviert, daher wird kein externer Widerstand benötigt.

## Installation

### 1. Voraussetzungen

- [ESPHome](https://esphome.io/guides/getting_started_command_line.html) installieren
- WiFi-Zugangsdaten bereithalten
- Impulse pro Kubikmeter Ihres Gaszählers kennen (siehe Zählerspezifikation)

### 2. Konfiguration

1. Repository klonen:

   ```bash
   git clone https://github.com/legacycode/ESPHome-Gas-Meter.git
   cd ESPHome-Gas-Meter
   ```

2. Eine `secrets.yaml` Datei aus der Vorlage erstellen:

   ```bash
   cp esphome/secrets.yaml.example esphome/secrets.yaml
   ```

   Dann `esphome/secrets.yaml` mit Ihren WiFi-Zugangsdaten bearbeiten:

   ```yaml
   wifi_ssid: "IhrWiFiSSID"
   wifi_password: "IhrWiFiPasswort"
   ```

3. Konfiguration in `esphome/gas-meter-wemos.yaml` anpassen:

   ```yaml
   substitutions:
     devicename: "gas-meter"
     friendly_name: "Gas Meter"
     pulses_per_cubic_meter: "100"  # An Ihren Zähler anpassen
     initial_meter_offset: "0"       # Auf Ihren aktuellen Zählerstand setzen
   ```

4. **(Optional)** Auf deutsche Lokalisierung umstellen:

   Zeile 50 in `esphome/gas-meter-wemos.yaml` ändern:
   ```yaml
   # Ändern von:
   translations: !include localization/en.yaml

   # Nach:
   translations: !include localization/de.yaml
   ```

### 3. Firmware flashen

**Flashen via USB:**

```bash
esphome run esphome/gas-meter-wemos.yaml
```

**Hinweis:** Dieses Projekt verwendet nur USB-Flashen. OTA-Updates sind
aus Gründen der Einfachheit und Sicherheit nicht enthalten.

## Alternative: Remote Packages verwenden

Anstatt das Repository zu klonen, können Sie die **Remote Package
Konfiguration** verwenden, die alle Dateien direkt von GitHub lädt. Dies ist
perfekt, wenn Sie keine lokalen Kopien der Konfigurationsdateien pflegen möchten.

### Schnelleinrichtung mit Remote Packages

1. **Voraussetzungen:**
   - [ESPHome](https://esphome.io/guides/getting_started_command_line.html) installieren
   - WiFi-Zugangsdaten bereithalten

2. **Nur die Remote-Config-Datei herunterladen:**

   ```bash
   curl -O https://raw.githubusercontent.com/legacycode/ESPHome-Gas-Meter/main/esphome/gas-meter-wemos-remote.yaml
   ```

3. **Eine secrets.yaml im selben Verzeichnis erstellen:**

   ```yaml
   wifi_ssid: "IhrWiFiSSID"
   wifi_password: "IhrWiFiPasswort"
   ```

4. **Konfigurationsvariablen** in `gas-meter-wemos-remote.yaml` anpassen:

   ```yaml
   substitutions:
     devicename: "gas-meter-remote"
     friendly_name: "Gas Meter Remote"
     pulses_per_cubic_meter: "100"  # An Ihren Zähler anpassen
     initial_meter_offset: "0"       # Auf Ihren aktuellen Zählerstand setzen
   ```

5. **Flashen via USB:**

   ```bash
   esphome run gas-meter-wemos-remote.yaml
   ```

### Wie Remote Packages funktionieren

Die Remote-Konfiguration lädt automatisch alle benötigten Dateien von
GitHub beim ersten Start:

- **common/packages.yaml** - Core ESPHome-Komponenten
- **common/boards/esp8266-d1-mini.yaml** - Board-Konfiguration
- **gas-meter/packages.yaml** - Alle Gaszähler-Funktionen
- **localization/en.yaml** - Englische Übersetzungen

Dateien werden lokal gecacht und alle 24 Stunden aktualisiert (`refresh: 1d`).

### Vorteile von Remote Packages

✅ **Kein Repository-Klonen** - Nur eine Config-Datei benötigt
✅ **Automatische Updates** - Erhalten Sie die neuesten Änderungen von GitHub
✅ **Minimale Wartung** - Keine lokale Dateiverwaltung
✅ **Gleiche Funktionalität** - 100% Funktionsparität mit lokaler Version

### Umstellung auf lokale Konfiguration

Wenn Sie lokale Dateien für Anpassungen bevorzugen, einfach:
1. Das vollständige Repository klonen
2. `esphome/gas-meter-wemos.yaml` statt Remote-Datei verwenden
3. Beliebige Paket-Dateien nach Bedarf modifizieren

## Konfigurationsoptionen

### Impulse pro Kubikmeter

Übliche Werte für Gaszähler:

- **1 Impuls/m³**: 1 Impuls = 1 Kubikmeter
- **10 Impulse/m³**: 1 Impuls = 0,1 Kubikmeter
- **100 Impulse/m³**: 1 Impuls = 0,01 Kubikmeter (am häufigsten)
- **1000 Impulse/m³**: 1 Impuls = 0,001 Kubikmeter

Prüfen Sie die Spezifikation Ihres Gaszählers oder den Zähler selbst für
den korrekten Wert.

### Anfangszählerstand-Offset

Setzen Sie dies auf Ihren aktuellen Gaszählerstand, um vom tatsächlichen
Zählerwert aus zu starten:

```yaml
initial_meter_offset: "1234"  # Ihr aktueller Zählerstand in m³
```

Sie können dies auch später über Home Assistant mit der Entität
**"Zaehlerstand-Offset"** anpassen.

## Home Assistant Integration

### Verfügbare Entitäten

Nach Hinzufügen des Geräts zu Home Assistant haben Sie Zugriff auf:

| Entität             | Typ           | Beschreibung                             |
|---------------------|---------------|------------------------------------------|
| **Flow Rate**       | Sensor        | Aktueller Gasdurchfluss (m³/h)           |
| **Total**           | Sensor        | Gemessener Verbrauch seit Reset (m³)     |
| **Meter Reading**   | Sensor        | Tatsächlicher Zählerstand (Offset+Total) |
| **Total Pulses**    | Sensor        | Rohe Impulszählung                       |
| **Meter Offset**    | Number        | Einstellbarer Offset zur Kalibrierung    |
| **Reset Pulses**    | Button        | Impulszähler auf Null zurücksetzen       |
| **Status**          | Binary Sensor | Gerätestatus online/offline              |
| **LED**             | Light         | Status-LED steuern                       |
| **WiFi Signal**     | Sensor        | WiFi-Signalstärke                        |
| **Uptime**          | Sensor        | Gerätebetriebszeit                       |

### Energie-Dashboard-Integration

1. Gehen Sie zu **Einstellungen** → **Dashboards** → **Energie**
2. Unter **Gasverbrauch**: **Gasquelle hinzufügen**
3. Wählen Sie: **sensor.gas_meter_meter_reading**
   (oder **sensor.gaszaehler_zaehlerstand** bei deutscher Lokalisierung)
4. Bestätigen

Das Energie-Dashboard verfolgt automatisch Ihren täglichen, monatlichen
und jährlichen Gasverbrauch.

### Jährliche Kalibrierung

Um mit der Ablesung Ihres Energieversorgers zu kalibrieren:

1. Vergleichen Sie den **Meter Reading** Sensor mit Ihrem physischen Zähler
2. Berechnen Sie die Differenz: `tatsaechlicher_stand - sensor_stand`
3. Passen Sie **Meter Offset** an, indem Sie die Differenz addieren
4. Der **Meter Reading** stimmt nun mit Ihrem physischen Zähler überein

**Beispiel:**

- Physischer Zähler: 1456 m³
- Sensor-Stand: 1450 m³
- Differenz: +6 m³
- Aktueller Offset: 1234 m³
- Neuer Offset: 1234 + 6 = **1240 m³**

## Architektur

Dieses Projekt verwendet eine **modulare paketbasierte Struktur** für
bessere Wartbarkeit:

```
esphome/
├── common/               # Basiskonfiguration
│   ├── boards/          # Board-spezifische Configs
│   ├── core/            # Core ESPHome-Komponenten
│   └── packages.yaml    # Aggregiert alle common-Pakete
├── gas-meter/           # Gaszähler-Funktionalität
│   ├── controls/        # Reset-Button, Offset-Nummer
│   ├── core/            # Boot, Globals, Pulse Meter Logik
│   ├── sensors/         # Diagnose-Sensoren
│   ├── led-internal.yaml
│   └── packages.yaml    # Aggregiert alle gas-meter-Pakete
├── localization/        # Sprachunterstützung (EN/DE)
└── gas-meter-wemos.yaml # Hauptgerätekonfiguration
```

Die Hauptkonfiguration beinhaltet nur 4 Pakete:
- `common/packages.yaml` - Core-Komponenten (esphome, wifi, api, preferences)
- Board-Konfiguration (ESP8266 D1 Mini)
- `gas-meter/packages.yaml` - Alle Gaszähler-Funktionen
- `localization/en.yaml` (oder de.yaml) - Sprachübersetzungen

## Fehlerbehebung

### Keine Impulse erkannt

1. Verkabelung prüfen (D2 und GND)
2. Position des Reed-Kontakts nahe dem Magneten überprüfen
3. Reed-Kontakt manuell mit einem Magneten testen
4. Logs prüfen: `esphome logs esphome/gas-meter-wemos.yaml`

### Impulse zu schnell/langsam

Interner Filter in `esphome/gas-meter/core/pulse-meter.yaml` anpassen:

```yaml
internal_filter: 200ms  # Von Standard 100ms erhöhen bei Fehlimpulsen
```

### Gerät verbindet sich nicht mit WiFi

1. `esphome/secrets.yaml` Zugangsdaten prüfen
2. Sicherstellen, dass 2,4 GHz WiFi verwendet wird (ESP8266 unterstützt kein 5 GHz)
3. WiFi-Signalstärke über Home Assistant Diagnose-Sensoren prüfen

**Hinweis:** Diese Konfiguration enthält keinen Fallback-WiFi-AP. Bei
fehlgeschlagener WiFi-Verbindung muss via USB mit korrigierten
Zugangsdaten neu geflasht werden.

## Was NICHT enthalten ist

Dies ist eine **vereinfachte Konfiguration** fokussiert auf Kernfunktionalität.
Die folgenden Features sind bewusst ausgeschlossen:

- ❌ OTA-Updates (USB zum Flashen verwenden)
- ❌ MQTT (stattdessen Home Assistant API verwenden)
- ❌ Web-Server (über Home Assistant konfigurieren)
- ❌ Captive Portal (kein Fallback-WiFi-AP)
- ❌ Zeit/NTP (nicht nötig für Impulszählung)
- ❌ API-Verschlüsselung (geeignet für vertrauenswürdige Heimnetzwerke)

**Warum vereinfacht?** Schnellere Kompilierung, kleinere Firmware, einfacher
zu verstehen und weniger Abhängigkeiten.

**Diese Features benötigt?** Sie können sie durch Erstellen zusätzlicher
Paket-Dateien in `esphome/common/core/` und deren Einbindung in
`gas-meter-wemos.yaml` hinzufügen.

## Continuous Integration

Dieses Projekt verwendet GitHub Actions zur automatischen Erstellung und
Validierung der Firmware bei jedem Push und Pull Request. Der Workflow:

- ✅ Kompiliert die YAML-Konfiguration
- ✅ Validiert die ESPHome-Konfigurationssyntax
- ✅ Stellt sicher, dass die Firmware erfolgreich erstellt wird

## Lizenz

Dieses Projekt ist unter der MIT-Lizenz lizenziert - siehe
[LICENSE](LICENSE) Datei für Details.

## Mitwirken

Beiträge sind willkommen! Bitte reichen Sie gerne einen Pull Request ein.

## Danksagungen

- Erstellt mit [ESPHome](https://esphome.io/)
- Kompatibel mit [Home Assistant](https://www.home-assistant.io/)

## Support

Bei Problemen oder Fragen öffnen Sie bitte ein Issue auf GitHub.

---

**Haftungsausschluss:** Dieses Projekt beinhaltet elektrische Komponenten
und Gaszähler-Modifikationen. Stellen Sie sicher, dass Sie lokale Vorschriften
und Sicherheitsstandards einhalten. Die Autoren übernehmen keine Verantwortung
für Schäden oder Verletzungen, die aus der Nutzung dieses Projekts resultieren.
