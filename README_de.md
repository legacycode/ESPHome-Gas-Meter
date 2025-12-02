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
- 📡 MQTT-Unterstützung für zusätzliche Integrationen
- 💾 Einstellbarer Zählerstand-Offset (für jährliche Kalibrierung)
- 🔄 Reset-Button für Impulszähler
- 💡 Visuelles LED-Feedback bei jedem Impuls
- 🌐 Web-Interface zur Konfiguration

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
   cp secrets.yaml.example secrets.yaml
   ```

   Dann `secrets.yaml` mit Ihren Zugangsdaten bearbeiten:

   ```yaml
   wifi_ssid: "IhrWiFiSSID"
   wifi_password: "IhrWiFiPasswort"
   ap_password: "FallbackAP"
   encryption_key: "ihr-32-zeichen-verschluesselungsschluessel"
   ota_password: "IhrOTAPasswort"
   ```

   Sichere Schlüssel generieren mit:
   ```bash
   # Für encryption_key (32 Bytes base64)
   openssl rand -base64 32

   # Für ota_password (Hex-String)
   openssl rand -hex 16
   ```

3. Konfiguration in `gas-meter-wemos-de.yaml` anpassen:

   ```yaml
   substitutions:
     pulses_per_cubic_meter: "100"  # An Ihren Zähler anpassen
     initial_meter_offset: "0"       # Auf Ihren aktuellen Zählerstand setzen
   ```

### 3. Firmware flashen

**Erstmaliges Flashen (via USB):**

```bash
esphome run gas-meter-wemos-de.yaml
```

**Over-the-Air-Updates (nach erstem Flash):**

```bash
esphome run gas-meter-wemos-de.yaml --device gas-meter.local
```

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

Nach dem Hinzufügen des Geräts zu Home Assistant haben Sie Zugriff auf:

| Entität                     | Typ           | Beschreibung                   |
|-----------------------------|---------------|--------------------------------|
| **Durchflussrate**          | Sensor        | Aktuelle Durchflussrate (m³/h) |
| **Gesamt**                  | Sensor        | Verbrauch seit Reset (m³)      |
| **Zaehlerstand**            | Sensor        | Zählerstand (Offset+Gesamt)    |
| **Gesamtimpulse**           | Sensor        | Rohe Impulszahl                |
| **Zaehlerstand-Offset**     | Number        | Offset zur Kalibrierung        |
| **Impulse zuruecksetzen**   | Button        | Zähler zurücksetzen            |
| **Status**                  | Binary Sensor | Gerätestatus                   |
| **LED**                     | Light         | Status-LED steuern             |
| **WiFi-Signal**             | Sensor        | WiFi-Signalstärke              |
| **Betriebszeit**            | Sensor        | Gerätebetriebszeit             |

### Energie-Dashboard Integration

1. Gehen Sie zu **Einstellungen** → **Dashboards** → **Energie**
2. Unter **Gasverbrauch**: **Gasquelle hinzufügen**
3. Wählen Sie: **sensor.gaszaehler_zaehlerstand**
4. Bestätigen

Das Energie-Dashboard erfasst automatisch Ihren täglichen, monatlichen und
jährlichen Gasverbrauch.

### Jährliche Kalibrierung

Zum Abgleich mit der Ablesung Ihres Energieversorgers:

1. Vergleichen Sie den **Zaehlerstand**-Sensor mit Ihrem physischen Zähler
2. Berechnen Sie die Differenz: `tatsaechlicher_stand - sensor_stand`
3. Passen Sie **Zaehlerstand-Offset** an, indem Sie die Differenz addieren
4. Der **Zaehlerstand** entspricht nun Ihrem physischen Zähler

**Beispiel:**

- Physischer Zähler: 1456 m³
- Sensor-Stand: 1450 m³
- Differenz: +6 m³
- Aktueller Offset: 1234 m³
- Neuer Offset: 1234 + 6 = **1240 m³**

## MQTT Integration

Das Gerät publiziert auf MQTT-Topics unter `esphome/gas-meter/`.

Home Assistant MQTT Discovery ist deaktiviert, wenn Sie die API verwenden:

```yaml
mqtt:
  discovery: false  # Bereits in der Konfiguration aktiviert
```

## Fehlersuche

### Keine Impulse erkannt

1. Verkabelung überprüfen (D2 und GND)
2. Prüfen, ob der Reed-Kontakt korrekt in der Nähe des Magneten positioniert ist
3. Reed-Kontakt manuell mit einem Magneten testen
4. Logs überprüfen: `esphome logs gas-meter-wemos-de.yaml`

### Impulse zu schnell/langsam

Internen Filter anpassen, um Fehlauslösungen zu verhindern:

```yaml
internal_filter: 100ms  # Erhöhen bei Fehlimpulsen
```

### Gerät verbindet sich nicht mit WiFi

1. `secrets.yaml` Zugangsdaten überprüfen
2. Fallback-AP verwenden: Mit "Gaszaehler Fallback" verbinden
3. WiFi über das Captive Portal konfigurieren

## Dateien

- `gas-meter-wemos-en.yaml` - Englische Konfiguration
- `gas-meter-wemos-de.yaml` - Deutsche Konfiguration (verwendet "ae",
  "oe", "ue" für MQTT-Kompatibilität)
- `secrets.yaml.example` - Vorlage für WiFi- und API-Zugangsdaten
- `secrets.yaml` - Ihre WiFi- und API-Zugangsdaten (nicht in git enthalten,
  aus Vorlage erstellen)
- `.github/workflows/build.yml` - GitHub Actions Workflow

## Continuous Integration

Dieses Projekt verwendet GitHub Actions, um die Firmware bei jedem Push und
Pull Request automatisch zu bauen und zu validieren. Der Workflow:

- ✅ Kompiliert beide YAML-Konfigurationen (Englisch und Deutsch)
- ✅ Validiert die ESPHome-Konfigurationssyntax
- ✅ Stellt sicher, dass die Firmware erfolgreich kompiliert

Der Workflow läuft auf dem `main`-Branch und verwendet die
`secrets.yaml.example` Vorlage zur Build-Validierung.

## Lizenz

Dieses Projekt ist unter der MIT-Lizenz lizenziert - siehe
[LICENSE](LICENSE) Datei für Details.

## Mitwirken

Beiträge sind willkommen! Bitte öffnen Sie gerne einen Pull Request.

## Danksagungen

- Erstellt mit [ESPHome](https://esphome.io/)
- Kompatibel mit [Home Assistant](https://www.home-assistant.io/)

## Support

Wenn Sie auf Probleme stoßen oder Fragen haben, öffnen Sie bitte ein Issue auf GitHub.

---

**Haftungsausschluss:** Dieses Projekt beinhaltet elektrische Komponenten und
Modifikationen am Gaszähler. Stellen Sie sicher, dass Sie lokale Vorschriften
und Sicherheitsstandards einhalten. Die Autoren sind nicht verantwortlich für
Schäden oder Verletzungen, die aus der Nutzung dieses Projekts resultieren.
