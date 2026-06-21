# ESPHome Voice Assistant

A modular ESPHome framework for voice assistant devices.

This root README is the setup and usage guide. For deep implementation details, see:
- [Technical Architecture and Device Extension Guide](ARCHITECTURE.md)

## Attribution

This project is based on the original work by **RealDeco** at [xiaozhi-esphome](https://github.com/RealDeco/xiaozhi-esphome/). The original monolithic configuration has been refactored into a modular architecture.

### Audio Credits

This project uses sounds from the [Home Assistant Voice Preview Edition](https://github.com/esphome/home-assistant-voice-pe) © 2024 by [Clayton Charles Tapp](https://www.cctaudio.com/), licensed under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).

## What This Project Provides

- Modular structure separating shared logic from device-specific hardware
- Event-driven voice assistant behavior for cleaner feature composition
- Native timer support with voice commands, display UI, audio feedback, and LED integration
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

## Recommended: Home Assistant Integration

Most users will integrate this project with Home Assistant's ESPHome add-on:

1. **Clone** this repository into your Home Assistant config folder:
   ```bash
   cd /config/esphome/
   git clone https://github.com/devbobo/esphome-voice-assistant.git
   ```

2. **Create a device configuration** in Home Assistant's ESPHome directory (e.g., `echo-ear.yaml`):
   ```yaml
   substitutions:
     name: echo-ear
     friendly_name: ESPHome Voice Assistant
     device: espressif/echoear

   packages:
     voice-assistant: !include esphome-voice-assistant/base.yaml

   wifi:
     ssid: !secret wifi_ssid
     password: !secret wifi_password
   ```

3. **Pull updates** to the project and rebuild on HA:
   ```bash
   cd /config/esphome/esphome-voice-assistant
   git pull
   ```
   Then rebuild the device in Home Assistant's ESPHome dashboard.

The modular architecture allows `base.yaml` to package all framework logic, while your device YAML adds instance-specific configuration (WiFi, device name, etc.).

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

Production-ready for:
- Voice assistant with wake-word detection
- Native timer management (set, update, cancel, display, audio, LED feedback)
- Gesture detection and touchscreen navigation
- Battery monitoring (EchoEar)
