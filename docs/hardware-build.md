# Hardware Build Guide

Complete hardware specifications and wiring guide for the GrandBoard ESPHome LED controller.

## Bill of Materials

| Component | Specification | Quantity | Notes |
|-----------|---------------|----------|-------|
| Microcontroller | Wemos D1 Mini (ESP8266) | 1 | 4MB flash recommended |
| LED Strip | BTF-LIGHTING WS2811 RGB COB | 1 | 12V, 70 addressable pixels |
| Level Shifter | 3.3V to 5V Logic Level Converter | 1 | Required for reliable data signal |
| Capacitor | 1000µF 16V Electrolytic | 1 | Across power supply, prevents voltage spikes |
| Resistor | 330Ω 1/4W | 1 | Inline with data line |
| Power Supply | 12V DC, 5A minimum | 1 | Sized for LED strip power draw |
| DC Barrel Jack | 5.5mm x 2.1mm | 1 | For power supply connection |
| Wire | 22 AWG stranded | ~2m | For connections |
| Heat Shrink | Assorted sizes | As needed | For wire protection |

## LED Strip Specifications

**Model:** BTF-LIGHTING FCOB WS2811 IC RGB COB LED Strip

| Specification | Value |
|---------------|-------|
| Voltage | 12V DC |
| LED Density | 630 LEDs/m (COB style) |
| IC Density | 14 ICs/m |
| Addressable Pixels | ~70 per 5m strip |
| Color Order | RGB |
| Data Protocol | WS2811 (400kHz) |
| Power Draw | ~18W/m at full white |
| Length Used | 5m (16.4ft) - fits dartboard perimeter |

## Wiring Diagram

```
                                    ┌─────────────────────┐
    12V Power Supply                │    LED Strip        │
    ================                │    (WS2811 12V)     │
         (+) ────────────┬──────────┤ VCC (12V)           │
                         │          │                     │
                    [1000µF]        │                     │
                    [16V  ]        │                     │
                         │          │                     │
         (-) ────────────┴──────────┤ GND                 │
                         │          │                     │
                         │          │                     │
    ┌────────────────────┘          │                     │
    │                               │                     │
    │    ┌─────────────────┐        │                     │
    │    │   D1 Mini       │        │                     │
    │    │   (ESP8266)     │        │                     │
    │    │                 │        │                     │
    │    │  GPIO2 (D4) ────┼──[330Ω]──┬──────────────────┤ Data In             │
    │    │                 │          │                   │                     │
    │    │                 │    ┌─────┴─────┐             │                     │
    │    │                 │    │  Level    │             │                     │
    │    │                 │    │  Shifter  │             │                     │
    │    │                 │    │           │             │                     │
    │    │            3.3V ├────┤ LV    HV  ├─────────────┘                     │
    │    │                 │    │           │                                   │
    └────┤ GND             ├────┤ GND   GND ├───────────────────────────────────┤ GND                 │
         │                 │    └───────────┘                                   │                     │
         │             5V  ├────────────────────────────────────────────────────┘
         │  (from USB or   │
         │   regulator)    │
         └─────────────────┘
```

## Simplified Wiring (Text)

```
D1 Mini Pin      Connection              Component           LED Strip
-----------      ----------              ---------           ---------
GPIO2 (D4)  -->  [330Ω Resistor]    -->  Level Shifter LV -> Level Shifter HV  -->  Data In
3.3V        -->  Level Shifter LV VCC
GND         -->  Level Shifter GND  -->  LED Strip GND  -->  Power Supply (-)
5V          -->  (optional - for logic)

Power Supply
------------
12V (+)     -->  [1000µF Cap (+)]   -->  LED Strip VCC (12V)
12V (-)     -->  [1000µF Cap (-)]   -->  LED Strip GND  -->  D1 Mini GND
```

## Component Details

### Level Shifter

**Why needed:** The D1 Mini outputs 3.3V logic signals, but WS2811 LEDs expect 5V logic. While some strips work with 3.3V, a level shifter ensures reliable communication.

**Recommended:**
- 74AHCT125 (best for single channel)
- TXS0108E bidirectional level shifter
- Simple BSS138 MOSFET level shifter module

**Wiring:**
- LV (Low Voltage) side: Connect to D1 Mini 3.3V
- HV (High Voltage) side: Connect to 5V (can tap from D1 Mini 5V pin or separate supply)
- GND: Common ground with D1 Mini and LED strip

### Capacitor

**Why needed:** The 1000µF capacitor across the power supply smooths voltage spikes when LEDs turn on/off rapidly, preventing damage and glitches.

**Specifications:**
- Capacitance: 1000µF (minimum 470µF)
- Voltage Rating: 16V or higher (for 12V supply)
- Type: Electrolytic

**Placement:** As close to LED strip power input as possible.

**Polarity:**
- Positive (+) stripe on capacitor to 12V+
- Negative (-) stripe to GND

### Resistor

**Why needed:** The 330Ω resistor on the data line prevents signal reflections and protects the first LED's data input from voltage spikes.

**Specifications:**
- Resistance: 330Ω (220Ω to 470Ω acceptable)
- Power Rating: 1/4W sufficient
- Type: Through-hole or SMD

**Placement:** Inline with data line, as close to LED strip as possible.

## Power Calculations

### LED Strip Power Draw

| Condition | Current Draw | Power (12V) |
|-----------|--------------|-------------|
| All OFF | ~20mA | 0.24W |
| All RED 100% | ~1.5A | 18W |
| All WHITE 100% | ~4.2A | 50W |
| Typical use (mixed colors) | ~1-2A | 12-24W |

### Power Supply Sizing

**Recommended:** 12V 5A (60W) power supply

This provides headroom for:
- Full brightness operation
- Startup surge current
- Long-term reliability

### D1 Mini Power

The D1 Mini can be powered via:
1. **USB** - For programming and low-power testing
2. **5V Pin** - From a 5V regulator or buck converter
3. **VIN Pin** - Accepts 5-12V (has onboard regulator)

**Note:** When using 12V LED strip power, use a buck converter to step down to 5V for the D1 Mini if not using USB power.

## Assembly Tips

### Soldering

1. **Tin wires first** - Apply solder to wire ends before joining
2. **Use flux** - Helps solder flow on LED strip pads
3. **Quick joints** - Don't overheat LED strip (max 3 seconds)
4. **Strain relief** - Hot glue or heat shrink at solder joints

### Capacitor Orientation

```
    Electrolytic Capacitor
    ┌─────────────────┐
    │  ─────────────  │  <-- Stripe indicates NEGATIVE (-)
    │  │           │  │
    │  │   1000µF  │  │
    │  │    16V    │  │
    │  │           │  │
    │  └───────────┘  │
    └────┬─────┬──────┘
         │     │
        (-)   (+)
         │     │
        GND   12V
```

### Level Shifter Module Wiring

```
    Level Shifter Module (typical 4-channel)
    ┌──────────────────────────────────────┐
    │  LV1  LV2  LV3  LV4    LV    GND     │  <-- Low voltage side (3.3V)
    │   │    │    │    │      │     │      │
    │   ○    ○    ○    ○      ○     ○      │
    │                                       │
    │   ○    ○    ○    ○      ○     ○      │
    │   │    │    │    │      │     │      │
    │  HV1  HV2  HV3  HV4    HV    GND     │  <-- High voltage side (5V)
    └──────────────────────────────────────┘

    Connections:
    - LV1: D1 Mini GPIO2 (D4) via 330Ω resistor
    - LV:  D1 Mini 3.3V
    - HV1: LED Strip Data In
    - HV:  5V supply
    - GND: Common ground (both sides)
```

## Enclosure Considerations

- Use a project box rated for the power supply heat
- Ensure ventilation for power supply and voltage regulator
- Keep D1 Mini antenna area clear for WiFi signal
- Use cable glands for weatherproofing if needed
- Mount near dartboard but away from dart impact zone

## Testing Procedure

1. **Visual inspection** - Check all solder joints, no shorts
2. **Continuity test** - Verify GND connections with multimeter
3. **Power test** - Apply power, check for smoke/heat (disconnect if any!)
4. **LED test** - Flash firmware, verify LEDs respond
5. **Color test** - Verify RGB order is correct (red shows as red)
6. **WiFi test** - Check signal strength in final mounting location
