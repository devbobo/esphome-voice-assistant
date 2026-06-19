# ESPHome Voice Assistant

A modular ESPHome framework for voice assistant devices.

This root README is the setup and usage guide. For deep implementation details, see:
- [Technical Architecture and Device Extension Guide](ARCHITECTURE.md)

## Attribution

This project is based on the original work by **RealDeco** at [xiaozhi-esphome](https://github.com/RealDeco/xiaozhi-esphome/). The original monolithic configuration has been refactored into a modular architecture.

## What This Project Provides

- Modular structure separating shared logic from device-specific hardware
- Event-driven voice assistant behavior for cleaner feature composition
- Reusable include packages for voice assistant, media player, sounds, and touchscreen handlers
- Internationalized labels and strings via substitutions

## Supported Devices

### EchoEar (ESP32-S3)

- 360x360 round touchscreen
- I2S microphone and speaker with codec support
- BQ27220 battery monitoring
- Gesture-capable touch input

Device details and hardware-specific setup:
- [EchoEar README](devices/espressif/echoear/README.md)

## Requirements

- ESPHome 2025.9.0+
- A supported ESP32 target device
- Home Assistant (recommended for full voice assistant workflow)

## Quick Start

1. Clone this repository.
2. Set Wi-Fi credentials in your `secrets.yaml`:

```yaml
wifi_ssid: "Your SSID"
wifi_password: "Your Password"
```

3. Confirm the active device path in `base.yaml`:

```yaml
substitutions:
  device: espressif/echoear
```

4. Build and flash:

```bash
esphome compile base.yaml
esphome flash base.yaml
esphome logs base.yaml
```

## Basic Configuration

### Device Selection

Use the `device` substitution in `base.yaml` to select the active device implementation:

```yaml
substitutions:
  device: espressif/echoear
```

### Sound URLs

Customize sound file locations in `config.yaml`:

```yaml
substitutions:
  xiaozhi_sounds: "https://github.com/RealDeco/xiaozhi-esphome/raw/main/sounds"
  sound_startup: "${xiaozhi_sounds}/Home_Connected.flac"
  sound_wake_word: "${xiaozhi_sounds}/wake_word_triggered.flac"
  sound_timer_finished: "${xiaozhi_sounds}/timer_finished.flac"
```

### Gesture Timing

Global gesture timing and orientation are set in `base.yaml` substitutions:

```yaml
substitutions:
  rotate_display: "0"
  long_press_duration: "500"
  double_press_window: "400"
```

## Internationalization

- Root-level strings: `i18n.yaml`
- Device-specific strings: `devices/<vendor>/<model>/i18n.yaml`

Update these substitutions to localize labels, gestures, and status text.

## Troubleshooting

- Verify ESPHome version is 2025.9.0+
- Confirm `base.yaml` points to a valid device folder
- Check serial logs with `esphome logs base.yaml`

For EchoEar-specific troubleshooting:
- [EchoEar Troubleshooting](devices/espressif/echoear/README.md#troubleshooting)

## Technical Documentation

For architecture internals, event flows, manager responsibilities, and a detailed new-device implementation checklist:
- [Technical Architecture and Device Extension Guide](ARCHITECTURE.md)

## Status

Production-ready for voice assistant, gesture detection, and battery monitoring. Additional display rendering and advanced timer UI integration are in progress.
