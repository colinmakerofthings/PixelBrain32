# 🧠✨ PixelBrain32

> A small brain for addressable LEDs.

PixelBrain32 is a personal ESP32-based controller for addressable LED strips, designed for use with ESPHome and Home Assistant.

It's a small custom PCB that powers an ESP32 from the same 5 V supply as the LEDs, provides proper signal conditioning and level shifting for WS2812-style strips, and exposes a physical button for local control.

---

## Features

- ESP32 dev board (socketed / swappable)
- Single 5 V power input for ESP32 and LED strip
- Designed for WS2812B / NeoPixel-compatible LEDs
- Proper LED data conditioning:
  - Series resistor
  - 5 V level shifter (74AHCT125)
  - Bulk power smoothing capacitor for LED inrush current
- Physical push button for ESPHome actions
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

## License

See [LICENSE](LICENSE) for details.
