# ESP32 Pi-Star Companion

A real-time DMR hotspot monitor built on an **ESP32 CYD**  
(Cheap Yellow Display — 320×240 TFT with touchscreen).

It connects via WebSocket to a lightweight Python server running on your **Pi-Star hotspot** and displays live DMR activity, last-heard stations, static talkgroups, and hotspot information — all on a standalone touchscreen device.

---

## 🚀 Features

- **Live DMR activity**
  - Callsign, name, country flag, talkgroup, slot, BER, RSSI, duration

- **Last heard list**
  - Scrollable list with callsign, name, talkgroup and local time

- **Static talkgroups**
  - Automatically fetched from BrandMeister API (with names)

- **Hotspot information page**
  - Operator, QTH, RX/TX frequencies, colour code, power, network, service status

- **Touch navigation**
  - Tap anywhere to cycle through pages

- **Clock synchronization**
  - Time and UTC offset initialized from Pi-Star

---

## 📺 Display Pages

| Page | Description |
|------|------------|
| **0 — Live** | Real-time QSO details + recent activity |
| **1 — Last Heard** | Last 10 stations |
| **2 — Static TGs** | Configured talkgroups with names |
| **3 — Hotspot Info** | Device and network configuration |

---

## 🧠 Architecture

```
Pi-Star (Raspberry Pi)
  └── monitor_mmdvm_ws.py
        ├── Reads MMDVM logs (live DMR events)
        ├── Parses hotspot configuration
        ├── Loads DMR ID database
        ├── Fetches BrandMeister talkgroups
        └── Sends JSON via WebSocket (port 8765)

ESP32 CYD
  └── Firmware (PlatformIO)
        ├── Connects to WebSocket server
        ├── Parses JSON messages
        └── Renders UI on TFT display
```

### JSON message types

- `snapshot` → configuration + static data (on connect / change)
- `live` → real-time DMR activity
- `heard_summary` → last heard stations

---

## 🧰 Hardware

| Component | Details |
|----------|--------|
| ESP32 | ESP32 Dev Module |
| Display | 2.8" ILI9341 TFT (320×240) |
| Touch | XPT2046 |
| Hotspot | Raspberry Pi running Pi-Star |

Target device:  
**ESP32-2432S028 ("Cheap Yellow Display")**

More info:  
https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/

---

## ⚙️ Installation

### 1. Flash the ESP32

- Use **PlatformIO (VS Code)**  
  or  
- Use the web flasher:  
  👉 https://esp32projects.myshack.ch/

---

### 2. Install the Pi-Star WebSocket server

SSH into your Pi-Star and run:

```bash
sudo wget -q -O /home/pi-star/monitor_mmdvm_ws.py \
  https://raw.githubusercontent.com/HB9IIU/ESP32-PI-STAR-CLIENT/main/InstallationFiles/monitor_mmdvm_ws.py \
  && sudo systemctl restart monitor_mmdvm_ws
```

---

## 📦 Dependencies

### ESP32 firmware

- TFT_eSPI
- XPT2046_Touchscreen
- arduinoWebSockets
- ArduinoJson v7
- espressif32 @ 6.9.0

---

### Pi-Star server

- Python 3.9+
- `websockets==13.1`

---

## 🔧 Service Management (Pi-Star)

```bash
sudo systemctl status monitor_mmdvm_ws
sudo systemctl restart monitor_mmdvm_ws
sudo journalctl -u monitor_mmdvm_ws -f
```

---

## 📄 License

MIT License

---

## 📡 Author

HB9IIU

```raw
Pi-Star (Raspberry Pi)
  └── monitor_mmdvm_ws.py   Python asyncio WebSocket server
        ├── tails MMDVM log for live DMR events
        ├── parses /etc/mmdvmhost for config (freq, CC, power…)
        ├── reads /etc/dmrgateway for hotspot DMR ID
        ├── looks up callsign names from DMRIds.dat
        ├── fetches static TG names from BrandMeister API
        └── broadcasts JSON over WebSocket on port 8765

ESP32 CYD (Cheap Yellow Display)
  └── main.cpp              Arduino / PlatformIO firmware
        ├── connects to Pi-Star WebSocket
        ├── parses snapshot + live JSON messages
        └── renders 4 touch-navigable display pages
```

**Two JSON message types** sent by the server:
- `snapshot` — hotspot config, operator info, static TGs, RadioID DB metadata (sent on connect and on config change)
- `live` — real-time DMR event (sent on each log line)
- `heard_summary` — last 10 heard stations (sent periodically)

---

## Hardware

| Component | Details |
|-----------|---------|
| ESP32 board | ESP32 Dev Module (esp32dev) |
| Display | 2.8" ILI9341 TFT, 320×240, SPI |
| Touch | XPT2046 resistive touchscreen |
| Hotspot | Pi-Star on Raspberry Pi (Bullseye) |

The project targets the **ESP32 CYD** (commonly sold as "ESP32-2432S028"), which integrates the display and touch controller on one board.
more info here: https://randomnerdtutorials.com/cheap-yellow-display-esp32-2432s028r/


---

## Installation & Configiuration

Step 1: use platformio in visual studio code to build & flash the cheap yello display, or flash directly with web flasher :https://esp32projects.myshack.ch/


Step 2. open a console on your Hpotpsot running PiStar and paste this.



```bash
sudo wget -q -O /home/pi-star/monitor_mmdvm_ws.py \
  https://raw.githubusercontent.com/HB9IIU/ESP32-PI-STAR-CLIENT/main/InstallationFiles/monitor_mmdvm_ws.py \
  && sudo systemctl restart monitor_mmdvm_ws
```

---

## Dependencies

### ESP32 firmware (PlatformIO)
- [TFT_eSPI](https://github.com/Bodmer/TFT_eSPI) — display driver
- [XPT2046_Touchscreen](https://github.com/PaulStoffregen/XPT2046_Touchscreen) — touch input
- [links2004/WebSockets](https://github.com/Links2004/arduinoWebSockets) — WebSocket client
- [ArduinoJson v7](https://arduinojson.org/) — JSON parsing
- espressif32 @ 6.9.0

### Pi-Star Python server
- Python 3.9+
- `websockets 13.1` (pinned — 14+ requires Python 3.11)
- Standard library only (asyncio, configparser, subprocess…)

---

## Service management on Pi-Star

```bash
sudo systemctl status  monitor_mmdvm_ws
sudo systemctl restart monitor_mmdvm_ws
sudo journalctl -u monitor_mmdvm_ws -f
```

---

## License

MIT — see [LICENSE](LICENSE) for details.

73 de HB9IIU