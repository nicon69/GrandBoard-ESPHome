# GrandBoard ESPHome LED Controller

ESPHome firmware for controlling WS2812 RGB LEDs around a GrandBoard 3s dartboard, with diyHue integration for score-reactive lighting via the GrandBoard app's native Philips Hue support.

## Features

- **WS2812 RGB LED Control** - Full color control with effects (Rainbow, Strobe, Pulse, etc.)
- **diyHue Integration** - Appears as a Philips Hue light to the GrandBoard app
- **Web Dashboard** - Built-in web interface for manual control and diagnostics
- **Home Assistant Compatible** - Native ESPHome integration
- **OTA Updates** - Update firmware wirelessly

## Hardware Requirements

| Component | Specification |
|-----------|---------------|
| Controller | Wemos D1 Mini (ESP8266) |
| LED Strip | WS2812/WS2811 RGB (70 addressable LEDs) |
| Data Pin | GPIO2 (D4) |
| Power | 5V or 12V depending on strip |

### Wiring

```
D1 Mini          LED Strip
--------         ---------
GPIO2 (D4) ----> Data In
GND        ----> GND
5V/VIN     ----> VCC (or external power supply)
```

**Recommended:** Add a 330Ω resistor on the data line and a 1000µF capacitor across power for stability.

## Network Configuration

| Service | Address |
|---------|---------|
| ESPHome Device | 10.0.12.194 |
| diyHue Bridge | 10.0.12.3 |
| Home Assistant | 10.0.12.3:8123 |

## Quick Start

### 1. Configure Secrets

Copy the secrets template and fill in your values:

```bash
cp esphome/secrets.yaml.example esphome/secrets.yaml
```

Edit `esphome/secrets.yaml`:
```yaml
wifi_ssid: "YourWiFiNetwork"
wifi_password: "YourWiFiPassword"
ota_password: "YourOTAPassword"
```

### 2. Flash Firmware

Using ESPHome CLI:
```bash
esphome run esphome/dartboard.yaml
```

Or via Home Assistant ESPHome addon.

### 3. Configure diyHue

The light should be auto-discovered. If not, add manually in diyHue with:
- Protocol: `esphome`
- IP: `10.0.12.194`
- Model: `ESPHome-RGB`

### 4. Pair GrandBoard App

1. Open GrandBoard app → Settings → Hue Lighting
2. Search for bridges → Select diyHue
3. Press link button (see below)
4. Select "Dartboard" light/room

**Link button command:**
```bash
curl -X PUT 'http://10.0.12.3/api/0/config' -d '{"linkbutton":true}'
```

## Web Interface

Access the dashboard at: **http://10.0.12.194**

Features:
- Color picker and brightness control
- LED effects (Rainbow, Strobe, Pulse, Flicker, Random)
- WiFi signal strength
- Device uptime
- Restart/Factory Reset buttons

## API Endpoints

| Endpoint | Description |
|----------|-------------|
| `/light/color_led` | Light state and control |
| `/text_sensor/light_id` | diyHue discovery info |
| `/sensor/WiFi%20Signal` | WiFi RSSI (dBm) |
| `/sensor/Uptime` | Device uptime |
| `/button/Restart/press` | Restart device |

> **Note:** ESPHome 2026.7.0+ requires entity names in URLs (spaces as `%20`).

### Control Examples

Turn on red:
```bash
curl -X POST "http://10.0.12.194/light/color_led/turn_on?r=255&g=0&b=0&brightness=255"
```

Turn off:
```bash
curl -X POST "http://10.0.12.194/light/color_led/turn_off"
```

Via diyHue (Hue API):
```bash
curl -X PUT -d '{"on":true,"bri":254,"xy":[0.64,0.33]}' \
  "http://10.0.12.3/api/<username>/lights/8/state"
```

## Troubleshooting

### Colors are wrong (e.g., red shows as green)

Change the color order in `dartboard.yaml`:
```yaml
light:
  - platform: neopixelbus
    type: RGB    # Try: RGB, GRB, BRG, RBG, GBR, BGR
```

### diyHue doesn't discover the light

1. Verify endpoints respond:
   ```bash
   curl http://10.0.12.194/text_sensor/light_id
   curl http://10.0.12.194/light/color_led
   ```

2. Check diyHue logs:
   ```bash
   docker logs diyhue | grep -i esphome
   ```

3. Restart diyHue:
   ```bash
   docker restart diyhue
   ```

### GrandBoard app dims lights to nothing

This was an issue with WLED + diyHue. The ESPHome integration resolves this.

## File Structure

```
grandboard-esphome/
├── README.md                 # This file
├── esphome/
│   ├── dartboard.yaml        # Main ESPHome configuration
│   ├── secrets.yaml          # Your secrets (git ignored)
│   └── secrets.yaml.example  # Secrets template
└── docs/
    └── setup-guide.md        # Detailed setup instructions
```

## License

MIT License - Feel free to use and modify.

## Acknowledgments

- [ESPHome](https://esphome.io/) - ESP8266/ESP32 firmware framework
- [diyHue](https://diyhue.org/) - Philips Hue bridge emulator
- [GrandBoard](https://www.gran-board.com/) - Electronic dartboard
