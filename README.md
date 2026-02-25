# 🧠✨ PixelBrain32

> A small brain for addressable LEDs.

PixelBrain32 is a personal ESP32-based controller for addressable LED strips, designed for use with ESPHome and Home Assistant.

It's a small custom PCB that powers an ESP32 from the same 5 V supply as the LEDs, provides proper signal conditioning
and level shifting for WS2812-style strips, and exposes a physical button for local control.

---

## Features

- ESP32 dev board (socketed / swappable)
- Single 5 V power input for ESP32 and LED strip
- Designed for WS2812B / NeoPixel-compatible LEDs
- Proper LED data conditioning:
  - Series resistor
  - 5 V level shifter (74AHCT125)
  - Bulk power smoothing capacitor for LED inrush current
- Two physical push buttons for local control:
  - SW1 (GPIO33): toggle on/off
  - SW2 (GPIO32): short press = brightness up, hold = brightness down
- Screw terminals for high-current LED connections
- Simple, reliable, and ESPHome-friendly

---

## Hardware Overview

| Component   | Details                          |
|-------------|----------------------------------|
| MCU         | ESP32 DevKit (plug-in headers)   |
| LED Power   | External 5 V PSU                 |
| LED Data    | 3.3 V → 5 V level-shifted        |
| Control     | ESPHome + Home Assistant         |
| Form factor | Custom PCB                       |

---

## Power & Safety Notes

- LED strips are powered directly from the external 5 V PSU
- ESP32 is powered from the same 5 V rail via its onboard regulator
- Bulk capacitance is used to protect LEDs from inrush current
- High-current paths are routed separately from logic signals

---

## Why This Exists

PixelBrain32 exists because:

- I wanted a clean, single-PSU LED controller
- I didn't want dangling dev boards or breadboards
- I wanted something reliable, understandable, and repairable
- I enjoy making things

---

## Circuit Diagram

```text
                          DC Jack (5 V in)
                               │
                         +─────┴─────+
                         │           │
                        +5V         GND
                         │           │
          ───────────────┼───────────┼───────────────────────────
                         │           │
                         ├──[1000µF]─┤   (bulk smoothing cap,
                         │  10 V     │    electrolytic, near
                         │           │    LED terminal)
                         │           │
                         ├───────────┴──────────────> LED GND (screw terminal)
                         │                            ESP32 GND
                         │
                         ├───────────────────────────> LED +5V (screw terminal)
                         │
                         ├───────────────────────────> ESP32 VIN (+5V)
                         │
                    74AHCT125
                  ┌──────────────┐
      +5V ───────>│ VCC          │
      GND ───────>│ GND          │
      GND ───────>│ /OE  (pin 1) │
                  │              │
ESP32 GPIO ──[330Ω]──> IN (pin 2)│
                  │  OUT (pin 3) ├──────────────────> LED DATA (screw terminal)
                  └──────────────┘
                  (100 nF bypass cap on VCC–GND, placed close to IC)

Push Button SW1 (Toggle on/off):
                         +3.3V (internal pull-up via INPUT_PULLUP)
                               │
ESP32 GPIO33 ─────────────────┤
                               │
                          [Button SW1]
                               │
                             GND
                    (100 nF C3 across button terminals for debounce)

Push Button SW2 (Brightness: short press = up, hold = down):
                         +3.3V (internal pull-up via INPUT_PULLUP)
                               │
ESP32 GPIO32 ─────────────────┤
                               │
                          [Button SW2]
                               │
                             GND
                    (100 nF C4 across button terminals for debounce)
```

### Component Reference

| Reference | Component            | Value / Part          | Notes                                      |
|-----------|----------------------|-----------------------|--------------------------------------------|
| J1        | DC Jack              | 5.5 / 2.1 mm barrel   | 5 V power input                            |
| J2        | Screw terminal (2P)  | —                     | LED +5V and GND output                     |
| J3        | Screw terminal (1P)  | —                     | LED DATA output                            |
| C1        | Electrolytic cap     | 1000 µF, 10 V         | Bulk smoothing, near LED terminal          |
| C2        | Ceramic cap          | 100 nF                | 74AHCT125 VCC bypass, place close to IC    |
| C3        | Ceramic cap          | 100 nF                | Across SW1 button terminals (debounce)     |
| C4        | Ceramic cap          | 100 nF                | Across SW2 button terminals (debounce)     |
| R1        | Resistor             | 330 Ω                 | Series data resistor, ESP32 → level shifter|
| U1        | Level shifter        | 74AHCT125             | 3.3 V → 5 V data signal conversion         |
| SW1       | Push button          | Tactile / momentary   | GPIO33, INPUT_PULLUP — toggle on/off       |
| SW2       | Push button          | Tactile / momentary   | GPIO32, INPUT_PULLUP — brightness control  |
| U2        | MCU                  | ESP32 DevKit          | Socketed on pin headers                    |

---

## ESPHome Configuration

```yaml
esphome:
  name: pixelbrain32
  friendly_name: PixelBrain32

esp32:
  board: esp32dev
  framework:
    type: arduino

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: !secret api_encryption_key

ota:
  - platform: esphome
    password: !secret ota_password

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "PixelBrain32 Fallback"
    password: !secret ap_password

captive_portal:

light:
  - platform: neopixelbus
    type: GRB
    variant: WS2812X
    pin: GPIO16          # adjust to your data output GPIO
    num_leds: 60         # adjust to your strip length
    name: "LED Strip"
    id: strip_light
    default_transition_length: 500ms
    restore_mode: ALWAYS_OFF   # prevents strip turning on after power loss or reboot

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO33
      mode: INPUT_PULLUP
      inverted: true
    name: "LED Toggle Button"
    on_press:
      then:
        - light.toggle: strip_light

  - platform: gpio
    pin:
      number: GPIO32
      mode: INPUT_PULLUP
      inverted: true
    name: "LED Brightness Button"
    on_multi_click:
      # Short press → brightness up 25%
      - timing:
          - ON for at most 350ms
          - OFF for at least 50ms
        then:
          - light.dim_relative:
              id: strip_light
              relative_brightness: 25%
              transition_length: 300ms
      # Long press → brightness down 25%
      - timing:
          - ON for at least 500ms
        then:
          - light.dim_relative:
              id: strip_light
              relative_brightness: -25%
              transition_length: 300ms
```

---

## License

See [LICENSE](LICENSE) for details.
