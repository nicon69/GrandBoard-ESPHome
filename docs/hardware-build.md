# Hardware Build Guide

Complete hardware specifications and wiring guide for the GrandBoard ESPHome LED controller.

## Bill of Materials

### Dartboard

| Component | Specification | Qty | Link |
|-----------|---------------|-----|------|
| Dartboard | GrandBoard 3s Electronic Dartboard | 1 | [Amazon](https://www.amazon.com/s?k=granboard+3s+dartboard&tag=rvpc-20) |

### Electronics

| Component | Specification | Qty | Link |
|-----------|---------------|-----|------|
| Microcontroller | Wemos D1 Mini (ESP8266) | 1 | [Amazon](https://www.amazon.com/s?k=wemos+d1+mini+esp8266&tag=rvpc-20) |
| LED Strip | BTF-LIGHTING WS2811 RGB COB 12V | 1 | [Amazon](https://www.amazon.com/s?k=BTF-LIGHTING+WS2811+COB+LED+strip+12V&tag=rvpc-20) |
| Level Shifter | 3.3V to 5V Logic Level Converter (4ch) | 1 | [Amazon](https://www.amazon.com/s?k=logic+level+converter+3.3v+5v+4+channel&tag=rvpc-20) |
| Capacitor | 1000µF 16V Electrolytic | 1 | [Amazon](https://www.amazon.com/s?k=1000uf+16v+electrolytic+capacitor&tag=rvpc-20) |
| Resistor Kit | 330Ω 1/4W (or assorted kit) | 1 | [Amazon](https://www.amazon.com/s?k=resistor+assortment+kit&tag=rvpc-20) |
| Power Supply | 12V DC 5A Adapter | 1 | [Amazon](https://www.amazon.com/s?k=12v+5a+power+supply+adapter+5.5mm&tag=rvpc-20) |
| DC Barrel Jack | 5.5mm x 2.1mm Female Panel Mount | 1 | [Amazon](https://www.amazon.com/s?k=dc+barrel+jack+panel+mount+5.5mm&tag=rvpc-20) |
| Proto Board | 5x7cm Double-Sided PCB | 1 | [Amazon](https://www.amazon.com/s?k=5x7cm+prototype+pcb+board&tag=rvpc-20) |
| Screw Terminals | 2-pin and 3-pin 5.08mm | 4 | [Amazon](https://www.amazon.com/s?k=pcb+screw+terminal+block+5.08mm&tag=rvpc-20) |
| Wire | 22 AWG Stranded Silicone | 1 | [Amazon](https://www.amazon.com/s?k=22+awg+silicone+wire&tag=rvpc-20) |
| Heat Shrink | Assorted Sizes Kit | 1 | [Amazon](https://www.amazon.com/s?k=heat+shrink+tubing+kit&tag=rvpc-20) |
| Project Box | 100x60x25mm ABS Enclosure | 1 | [Amazon](https://www.amazon.com/s?k=project+box+enclosure+abs+100x60&tag=rvpc-20) |

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

## PCB Assembly

Using a prototype PCB board creates a cleaner, more reliable build than loose wiring.

### PCB Layout (5x7cm board)

```
    ┌─────────────────────────────────────────────────────────┐
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ┌─────────────────────┐ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ │                     │ ○ ○ ┌───┐ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ │     D1 Mini         │ ○ ○ │330│ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ │    (socket)         │ ○ ○ │ Ω │ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ │                     │ ○ ○ └───┘ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ └─────────────────────┘ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ┌─────────────┐ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ │Level Shifter│ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ └─────────────┘ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ [====CAP====] ○ ○  │
    │  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○  │
    │                                                         │
    │  [12V IN]        [GND]        [LED OUT]                │
    │   ┌──┬──┐       ┌──┬──┐      ┌──┬──┬──┐                │
    │   │+ │- │       │G │G │      │V │D │G │                │
    │   └──┴──┘       └──┴──┘      └──┴──┴──┘                │
    └─────────────────────────────────────────────────────────┘

    Screw Terminals:
    - 12V IN: Power supply input (+12V, GND)
    - GND: Extra ground connection point
    - LED OUT: To LED strip (VCC, Data, GND)
```

### Component Placement

1. **D1 Mini** - Use female header sockets so the board is removable
2. **Level Shifter** - Mount near the D1 Mini, close to GPIO2
3. **330Ω Resistor** - Between level shifter output and LED terminal
4. **1000µF Capacitor** - Near the 12V input terminal, observe polarity!
5. **Screw Terminals** - At board edge for easy wire connections

### Soldering Order

1. Screw terminals (lowest profile)
2. Resistor
3. Female headers for D1 Mini
4. Level shifter (or headers for it)
5. Capacitor (tallest component)
6. Wire bridges on back of PCB

### PCB Wiring (Back Side)

Use short wire jumpers on the back of the PCB to connect:

```
12V+ ──────────────────────────> LED VCC terminal
      └──> Capacitor (+)

GND ───┬──> D1 Mini GND
       ├──> Level Shifter GND (both sides)
       ├──> Capacitor (-)
       └──> LED GND terminal

D1 Mini 3.3V ──> Level Shifter LV

D1 Mini 5V ────> Level Shifter HV

D1 Mini GPIO2 (D4) ──> Level Shifter LV1 input

Level Shifter HV1 output ──> 330Ω ──> LED Data terminal
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
