# ESP32 Pi-Star Companion

A real-time DMR hotspot monitor built on an **ESP32 CYD** (Cheap Yellow Display — 320×240 TFT with touchscreen).
It connects via WebSocket to a lightweight Python server running on your **Pi-Star** hotspot and displays live DMR activity, last-heard stations, static talkgroups, and hotspot configuration — all on a colour touchscreen.

---

## Features

- **Live DMR activity** — callsign, name, flag, talkgroup, slot, BER, RSSI, duration
- **Last heard list** — scrollable rows with flag, callsign, first name, talkgroup, and local time (HH:MM)
- **Static talkgroups** — fetched automatically from the BrandMeister API with names
- **Hotspot info page** — operator, QTH, RX/TX frequencies, colour code, power, network, service state
- **Touch navigation** — tap anywhere to cycle through the 4 pages
- **Country flags** — small pixel-art flags for every heard station
- **Automatic reconnection** — WebSocket reconnects and ARP warmup on disconnect
- **Clock sync** — UTC offset and time initialized from the Pi-Star snapshot

---

## Pages

| Page | Content |
|------|---------|
| 0 — Live | Real-time QSO: flag, callsign, name, location, TG, BER, RSSI, duration, recent-heard row |
| 1 — Last Heard | Last 10 stations: flag, callsign, name, talkgroup, local time |
| 2 — Static TGs | Static talkgroups configured on the hotspot with BrandMeister names |
| 3 — Hotspot Info | Operator, QTH, RX/TX freq, colour code, power, DMR network, service state |

---

## Architecture

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

---

## Installation

### 1 — Pi-Star side (one-liner)
metadata
source
#define WIFI_SSID "your-wifi-ssid"
#define WIFI_PASS "your-wifi-password"
#define WS_HOST   "192.168.x.x"   // local IP of your Pi-Star hotspot

> `secrets.h` is listed in `.gitignore` and will never be committed.

4. Build and flash:

```bash
pio run --target upload
```

Each normal build also creates a merged image automatically here:

```text
firmware/firmware.bin
```

So after a plain:

```bash
pio run -e esp32dev
```

you will always have a fresh merged `firmware.bin` ready to use.

---

## Updating the Python server

After pulling changes, push to GitHub then update Pi-Star in one command:

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