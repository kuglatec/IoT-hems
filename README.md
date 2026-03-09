# IoT-hems

Ein Schulprojekt im Informatikunterricht der **HEMS**-Schule. Das System nutzt einen **Raspberry Pi** als zentralen Host und **ESP32**-Mikrocontroller als IoT-Clients. Es unterstützt Temperaturanzeige, Lichtsteuerung und optional eine Kamera.

---

## Inhaltsverzeichnis

- [Überblick](#überblick)
- [Hardwareanforderungen](#hardwareanforderungen)
- [Systemarchitektur](#systemarchitektur)
- [Funktionen](#funktionen)
- [Inbetriebnahme](#inbetriebnahme)
  - [Raspberry Pi einrichten](#raspberry-pi-einrichten)
  - [ESP32-Client einrichten](#esp32-client-einrichten)
- [Projektstruktur](#projektstruktur)
- [Lizenz](#lizenz)

---

## Überblick

IoT-hems verbindet einen oder mehrere ESP32-Geräte über WLAN mit einem zentralen Raspberry Pi. Jeder ESP32-Knoten kann seine Umgebung erfassen (Temperatur, Lichtverhältnisse) und Aktoren steuern (LEDs, Relais für Beleuchtung). Optional kann ein Kameramodul am Raspberry Pi oder als ESP32-CAM zur visuellen Überwachung eingesetzt werden. Dieses Projekt wurde im Rahmen des Informatikunterrichts an der HEMS-Schule entwickelt.

---

## Hardwareanforderungen

### Host (Raspberry Pi)
- Raspberry Pi 3 / 4 / Zero 2 W (oder neuer)
- MicroSD-Karte (mindestens 8 GB, Klasse 10 empfohlen)
- Netzteil (5 V / 3 A)
- WLAN-Verbindung (integriert oder per USB-Adapter)

### IoT-Clients (ESP32)
- ESP32-Entwicklungsboard (z. B. ESP32-DevKitC, ESP32-WROOM-32)
- Temperatursensor (z. B. DHT22 / DS18B20 / BME280)
- LED oder Relaismodul zur Lichtsteuerung
- *(Optional)* ESP32-CAM-Modul für Kameraunterstützung

---

## Systemarchitektur

```
┌──────────────────────────────────────┐
│         Raspberry Pi (Host)          │
│  - MQTT-Broker (z. B. Mosquitto)     │
│  - Dashboard / Web-Oberfläche        │
│  - Optional: Kamera-Stream           │
└────────────────┬─────────────────────┘
                 │ WLAN / MQTT
      ┌──────────┴──────────┐
      │                     │
┌─────▼──────┐       ┌──────▼─────┐
│  ESP32 #1  │  ...  │  ESP32 #N  │
│ - Temp     │       │ - Temp     │
│ - Licht    │       │ - Licht    │
│ - (Kamera) │       │            │
└────────────┘       └────────────┘
```

Der Raspberry Pi betreibt einen MQTT-Broker. Jeder ESP32-Client verbindet sich mit dem Broker, veröffentlicht Sensordaten und abonniert Steuerungsbefehle für Aktoren.

---

## Funktionen

| Funktion              | Beschreibung                                                                                              |
|-----------------------|-----------------------------------------------------------------------------------------------------------|
| Temperaturanzeige     | Jeder ESP32 liest einen Temperatursensor aus und sendet die Messwerte per MQTT. Der Raspberry Pi sammelt und zeigt die Werte auf einem Dashboard an. |
| Lichtsteuerung        | ESP32-Knoten steuern LEDs oder relaisgeschaltete Lampen über MQTT-Befehle des Hosts.                      |
| Kamera (optional)     | Ein ESP32-CAM oder eine USB-/CSI-Kamera am Raspberry Pi kann Bilder streamen oder aufnehmen.              |

---

## Inbetriebnahme

### Raspberry Pi einrichten

1. **Raspberry Pi OS** (Lite oder Desktop) mit dem [Raspberry Pi Imager](https://www.raspberrypi.com/software/) auf die SD-Karte installieren.

2. **WLAN und SSH** in den Imager-Einstellungen oder über `raspi-config` aktivieren.

3. **MQTT-Broker installieren** (Mosquitto):
   ```bash
   sudo apt update && sudo apt install -y mosquitto mosquitto-clients
   sudo systemctl enable mosquitto
   sudo systemctl start mosquitto
   ```

4. *(Optional)* **Dashboard installieren**, z. B. Node-RED oder Home Assistant, um Sensordaten anzuzeigen und Geräte zu steuern.

### ESP32-Client einrichten

1. [Arduino IDE](https://www.arduino.cc/en/software) oder [PlatformIO](https://platformio.org/) installieren.

2. ESP32-Board-Unterstützung hinzufügen:
   - Arduino IDE: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json` unter *Einstellungen → Zusätzliche Boardverwalter-URLs* eintragen, dann **esp32** über den Board-Manager installieren.

3. Benötigte Bibliotheken installieren (über den Arduino-Bibliotheksverwalter oder PlatformIO):
   - `PubSubClient` – MQTT-Client
   - `DHT sensor library` (oder passende Bibliothek für den verwendeten Sensor)
   - `WiFi` (im ESP32-Arduino-Core enthalten)

4. `src/clients/client.ino` öffnen, WLAN-Zugangsdaten und die IP-Adresse des Raspberry Pi eintragen und den Sketch auf den ESP32 hochladen.

5. Nach dem Hochladen verbindet sich der ESP32:
   - mit dem WLAN
   - mit dem MQTT-Broker auf dem Raspberry Pi
   - veröffentlicht Temperaturwerte unter `hems/<geraete_id>/temperatur`
   - abonniert `hems/<geraete_id>/licht` für Lichtbefehle (`AN` / `AUS`)

---

## Projektstruktur

```
IoT-hems/
├── src/
│   └── clients/
│       └── client.ino   # Arduino-Sketch für ESP32-IoT-Clients
├── LICENSE
└── README.md
```

---

## Lizenz

Dieses Projekt steht unter den Bedingungen der [LICENSE](LICENSE)-Datei in diesem Repository.
