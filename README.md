# GrandBoard ESPHome LED Controller

ESPHome firmware for controlling WS2812/WS2811 RGB LEDs around a GrandBoard 3s dartboard, with diyHue integration for score-reactive lighting via the GrandBoard app's native Philips Hue support.

## Features

- **WS2812/WS2811 RGB LED Control** - Full color control with effects (Rainbow, Strobe, Pulse, etc.)
- **diyHue Integration** - Appears as a Philips Hue light to the GrandBoard app
- **Web Dashboard** - Built-in web interface for manual control and diagnostics
- **Home Assistant Compatible** - Native ESPHome integration
- **OTA Updates** - Update firmware wirelessly

## Hardware Requirements

| Component | Specification |
|-----------|---------------|
| Controller | Wemos D1 Mini (ESP8266) |
| LED Strip | WS2811 RGB 12V (~70 addressable pixels) |
| Level Shifter | 3.3V to 5V logic converter |
| Capacitor | 1000µF 16V electrolytic |
| Resistor | 330Ω on data line |
| Power Supply | 12V 5A DC |
| Data Pin | GPIO2 (D4) |

See [docs/hardware-build.md](docs/hardware-build.md) for complete build guide with wiring diagrams.

### Wiring Overview

```
D1 Mini GPIO2 (D4) --[330Ω]-- Level Shifter LV --> HV --> LED Data In
D1 Mini GND -----------------------------------------> LED GND --> Power Supply (-)
Power Supply 12V (+) --[1000µF Cap]----------------> LED VCC
```

## Network Configuration

Configure these addresses in `dartboard.yaml` for your network:

| Service | Example Address | Description |
|---------|-----------------|-------------|
| ESPHome Device | `192.168.1.100` | Static IP for the LED controller |
| diyHue Bridge | `192.168.1.50` | Your diyHue Docker host |
| Gateway | `192.168.1.1` | Your router |

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

### 2. Configure Network

Edit `esphome/dartboard.yaml` and update the IP addresses for your network:

```yaml
wifi:
  manual_ip:
    static_ip: 192.168.1.100  # Choose an unused IP on your network
    gateway: 192.168.1.1       # Your router IP
    subnet: 255.255.255.0
```

### 3. Flash Firmware

Using ESPHome CLI:
```bash
esphome run esphome/dartboard.yaml
```

Or via Home Assistant ESPHome addon.

### 4. Configure diyHue

The light should be auto-discovered. If not, add manually in diyHue with:
- Protocol: `esphome`
- IP: `<your-device-ip>`
- Model: `ESPHome-RGB`

### 5. Pair GrandBoard App

1. Open GrandBoard app → Settings → Hue Lighting
2. Search for bridges → Select diyHue
3. Press link button (see below)
4. Select "Dartboard" light/room

**Link button command:**
```bash
curl -X PUT 'http://<diyhue-ip>/api/0/config' -d '{"linkbutton":true}'
```

## Web Interface

Access the dashboard at: **http://\<device-ip\>**

Features:
- Color picker and brightness control
- LED effects (Rainbow, Strobe, Pulse, Flicker, Random)
- WiFi signal strength
- Device uptime and IP address
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
curl -X POST "http://<device-ip>/light/color_led/turn_on?r=255&g=0&b=0&brightness=255"
```

Turn off:
```bash
curl -X POST "http://<device-ip>/light/color_led/turn_off"
```

Via diyHue (Hue API):
```bash
curl -X PUT -d '{"on":true,"bri":254,"xy":[0.64,0.33]}' \
  "http://<diyhue-ip>/api/<username>/lights/<light-id>/state"
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
   curl http://<device-ip>/text_sensor/light_id
   curl http://<device-ip>/light/color_led
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

### LEDs flicker or show wrong colors randomly

- Check level shifter connections
- Verify 330Ω resistor is on data line
- Ensure capacitor is across power supply
- Check for loose solder joints

## Documentation

| Document | Description |
|----------|-------------|
| [README.md](README.md) | This file - overview and quick start |
| [docs/setup-guide.md](docs/setup-guide.md) | Detailed setup instructions |
| [docs/hardware-build.md](docs/hardware-build.md) | Hardware specs, wiring diagrams, BOM |

## File Structure

```
grandboard-esphome/
├── README.md                 # Overview and quick start
├── esphome/
│   ├── dartboard.yaml        # Main ESPHome configuration
│   ├── secrets.yaml          # Your secrets (git ignored)
│   └── secrets.yaml.example  # Secrets template
└── docs/
    ├── setup-guide.md        # Detailed setup instructions
    └── hardware-build.md     # Hardware build guide
```

## License

MIT License - Feel free to use and modify.

## Acknowledgments

- [ESPHome](https://esphome.io/) - ESP8266/ESP32 firmware framework
- [diyHue](https://diyhue.org/) - Philips Hue bridge emulator
- [GrandBoard](https://www.gran-board.com/) - Electronic dartboard
