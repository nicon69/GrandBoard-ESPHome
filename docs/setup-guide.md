# GrandBoard ESPHome Setup Guide

Complete setup instructions for the GrandBoard dartboard LED controller with diyHue integration.

## Prerequisites

- GrandBoard 3s dartboard with LED strip installed
- Wemos D1 Mini (ESP8266) controller
- WS2812/WS2811 RGB LED strip (70 addressable LEDs)
- diyHue bridge running (Docker recommended)
- Home Assistant with ESPHome addon (optional but recommended)

## Hardware Setup

### Components

| Part | Purpose |
|------|---------|
| Wemos D1 Mini | Microcontroller |
| WS2812 Strip | RGB LEDs |
| 330Ω Resistor | Data line protection |
| 1000µF Capacitor | Power smoothing |
| Level Shifter | 3.3V to 5V signal (optional) |

### Wiring Diagram

```
                    ┌─────────────┐
                    │  D1 Mini    │
                    │             │
    LED Strip       │    D4/GPIO2 ├───[330Ω]───> Data In
    ─────────       │         GND ├─────────────> GND
                    │         5V  ├─────────────> VCC
                    └─────────────┘
                                        │
                              [1000µF Cap]
                                        │
                                       GND
```

**Notes:**
- GPIO2 (D4) uses UART1 for reliable LED timing
- Logger baud_rate set to 0 to free UART1
- External 5V power recommended for >30 LEDs

## Software Setup

### Step 1: Install ESPHome

**Option A: Home Assistant Addon (Recommended)**
1. Go to Supervisor → Add-on Store
2. Install "ESPHome"
3. Start the addon

**Option B: Command Line**
```bash
pip install esphome
```

### Step 2: Configure Secrets

1. Copy the secrets template:
   ```bash
   cp esphome/secrets.yaml.example esphome/secrets.yaml
   ```

2. Edit `esphome/secrets.yaml`:
   ```yaml
   wifi_ssid: "YourWiFiNetwork"
   wifi_password: "YourWiFiPassword"
   ota_password: "ChooseASecurePassword"
   ```

### Step 3: Customize Configuration

Edit `esphome/dartboard.yaml` if needed:

**Change LED count:**
```yaml
light:
  - platform: neopixelbus
    num_leds: 70  # Change to match your strip
```

**Change IP address:**
```yaml
wifi:
  manual_ip:
    static_ip: <device-ip>  # Change to your network
    gateway: 10.0.12.1
    subnet: 255.255.255.0
```

**Change color order (if colors appear wrong):**
```yaml
light:
  - platform: neopixelbus
    type: RGB  # Options: RGB, GRB, BRG, RBG, GBR, BGR
```

### Step 4: Flash Firmware

**First flash (USB required):**
```bash
esphome run esphome/dartboard.yaml
```

Select the USB serial port when prompted.

**Subsequent updates (OTA):**
```bash
esphome run esphome/dartboard.yaml --device <device-ip>
```

Or use the ESPHome dashboard in Home Assistant.

### Step 5: Verify Device

1. Open http://<device-ip> in browser
2. You should see the ESPHome dashboard with:
   - Light controls (on/off, color, effects)
   - WiFi signal strength
   - Device uptime
   - Restart button

## diyHue Configuration

### Automatic Discovery

diyHue should auto-discover the ESPHome device. Check logs:
```bash
docker logs diyhue | grep -i esphome
```

You should see:
```
ESPHome: Found dartboard-leds at ip <device-ip>
ESPHome: dartboard-leds is a RGB ESPHome device
```

### Manual Configuration

If auto-discovery fails, edit the diyHue lights config:

```bash
docker exec diyhue cat /opt/hue-emulator/config/lights.yaml
```

Add or modify the entry:
```yaml
'8':
  name: Dartboard
  modelid: LCT015
  protocol: esphome
  protocol_cfg:
    ip: <device-ip>
    mac: a4:cf:12:be:3d:d1
    name: dartboard-leds
    esphome_model: ESPHome-RGB
    ct_boost: '0'
    rgb_boost: '0'
```

Restart diyHue:
```bash
docker restart diyhue
```

## GrandBoard App Setup

### Initial Pairing

1. Open GrandBoard app
2. Go to **Settings** → **Hue Lighting**
3. Tap **Search for bridges**
4. Select the diyHue bridge (shows as Philips Hue)

### Press Link Button

When the app asks you to press the bridge link button:

```bash
curl -X PUT 'http://<diyhue-ip>/api/0/config' -d '{"linkbutton":true}'
```

You have 30 seconds to complete pairing after this command.

### Select Light

1. After pairing, select **Dartboard** from the light list
2. Configure your desired effects for:
   - Single scores
   - Double scores
   - Triple scores
   - Bullseye

## Testing

### Test via ESPHome Dashboard

1. Go to http://<device-ip>
2. Use the color picker to change colors
3. Try different effects (Rainbow, Strobe, Pulse)

### Test via diyHue API

```bash
# Turn on red
curl -X PUT -d '{"on":true,"bri":254,"xy":[0.64,0.33]}' \
  "http://<diyhue-ip>/api/<api-username>/lights/8/state"

# Turn on green
curl -X PUT -d '{"on":true,"bri":254,"xy":[0.17,0.7]}' \
  "http://<diyhue-ip>/api/<api-username>/lights/8/state"

# Turn on blue
curl -X PUT -d '{"on":true,"bri":254,"xy":[0.15,0.06]}' \
  "http://<diyhue-ip>/api/<api-username>/lights/8/state"

# Turn off
curl -X PUT -d '{"on":false}' \
  "http://<diyhue-ip>/api/<api-username>/lights/8/state"
```

### Test with Darts

1. Start a game in GrandBoard app
2. Throw darts at the board
3. LEDs should react based on your Hue Lighting settings

## Troubleshooting

### Device not responding

1. Check power connections
2. Verify WiFi credentials in secrets.yaml
3. Check if device created fallback AP: "Dartboard-LEDs"
4. Connect to fallback AP and access http://192.168.4.1

### Wrong colors

Change color order in dartboard.yaml:
- WS2812B typically uses GRB
- WS2811 typically uses RGB
- Try different options until colors match

### diyHue shows "unreachable"

1. Verify ESPHome is responding: `curl http://<device-ip>/light/color_led`
2. Check diyHue protocol is set to `esphome` (not `native_multi`)
3. Restart diyHue: `docker restart diyhue`

### GrandBoard app doesn't find bridge

1. Ensure phone is on same network as diyHue
2. Check diyHue is running: `docker ps | grep diyhue`
3. Verify diyHue responds: `curl http://<diyhue-ip>/api/nouser/config`

### Lights dim to nothing when scoring

This was an issue with WLED firmware. ESPHome integration resolves this by using the native ESPHome protocol instead of the WLED/native_multi protocol.

## Maintenance

### Updating Firmware

```bash
esphome upload esphome/dartboard.yaml --device <device-ip>
```

### Viewing Logs

```bash
esphome logs esphome/dartboard.yaml --device <device-ip>
```

### Factory Reset

Access http://<device-ip> and click "Factory Reset" button, or:
```bash
curl -X POST http://<device-ip>/button/factory_reset/press
```

## Useful Commands

```bash
# ESPHome device status
curl http://<device-ip>/light/color_led

# diyHue lights list
curl http://<diyhue-ip>/api/<api-username>/lights

# diyHue logs
docker logs diyhue -f

# Restart diyHue
docker restart diyhue

# Press link button
curl -X PUT 'http://<diyhue-ip>/api/0/config' -d '{"linkbutton":true}'
```
