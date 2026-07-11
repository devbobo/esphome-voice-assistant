# Matrix Voice ESP Device Configuration

Matrix Voice is a 8-microphone array with LED ring feedback, speaker output, and optional buttons. This configuration provides a complete ESPHome voice assistant setup with LED visual feedback and microphone mute control.

## Hardware Overview

- **Microcontroller**: ESP-WROOM-32 (Espressif ESP32)
  - Dual-core 240MHz processor
  - 4MB Flash, 520KB RAM
  - Built-in WiFi and Bluetooth

- **FPGA**: Lattice Semiconductor
  - Manages all I/O multiplexing
  - 8 MEMS Microphones (MP34DB02) → mixed mono stream
  - 3W Stereo Class-D Amplifier (PAM8019) control
  - 18 RGBW Addressable LED ring control
  - Connected via Wishbone protocol over SPI

- **Audio Input**: 8 MEMS Microphones (MP34DB02)
  - Connected via FPGA
  - 16kHz, 16-bit PCM audio stream
  - Beamforming support (via FPGA)

- **Audio Output**: 3W Stereo Class-D Amplifier (PAM8019)
  - 3.5mm audio jack
  - 16kHz, 16-bit PCM input via FPGA
  - Volume control (0-100%)

- **LED Ring**: 18 RGBW Addressable LEDs (APA102 or compatible)
  - FPGA-controlled via matrixio platform
  - 4 channels per LED (R, G, B, White)
  - Visual feedback for device state

- **Input**: Mute button (optional)

## Module Structure

### Device Configuration Files

| File | Purpose |
|------|---------|
| `device.yaml` | Orchestrator - loads all device-specific modules; configures external matrixio component |
| `hardware.yaml` | ESP32 base configuration, FPGA SPI bus setup, matrixio component initialization |
| `io_entities.yaml` | High-level entities: LED ring, microphone, speaker (all via matrixio FPGA interface) |
| `led_ring.yaml` | LED state manager with event-driven feedback |
| `i18n.yaml` | Device-specific display labels |

### FPGA Communication (SPI/Wishbone)

Matrix Voice communicates with the FPGA via Wishbone protocol over SPI. The gnumpi matrixio component handles all register access and multiplexing.

| Signal | GPIO | Direction | Purpose |
|--------|------|-----------|---------|
| **CLK** | GPIO32 | → | FPGA SPI clock (8MHz) |
| **MOSI** | GPIO33 | → | ESP32 to FPGA data |
| **MISO** | GPIO21 | ← | FPGA to ESP32 data |
| **CS** | GPIO23 | → | Chip select (active low) |

## LED Ring State Feedback

The LED ring provides visual feedback for device state:

| State | Animation | Color |
|-------|-----------|-------|
| **Listening** | Pulse | Blue |
| **Muted** | Solid | Red |
| **Processing** | Pulse | Green |
| **Idle** | Off | — |
| **Error** | Strobe | Red |

## Audio Event Flow

```
User Voice Input
    ↓
Microphone (FPGA → matrixio) → ESP32
    ↓
Device-scoped voice_assistant package
    ↓
event.trigger → system_event
    ↓
event_router.yaml routes events to:
  - managers/audio.yaml
  - managers/voice.yaml
  - managers/led_ring.yaml
    ↓
LED ring manager updates light.led_ring via matrixio
    ↓
Update LED ring color/animation via matrixio platform
```

## Configuration

### Device Selection (base.yaml)

```yaml
substitutions:
  device: matrix/matrix-voice-esp  # Selects this device's config
```

### Feature Flags (device.yaml)

Enable/disable features for multi-device support:

```yaml
substitutions:
  enable_audio_manager: "true"       # Audio event coordination
  enable_led_manager: "true"         # LED visual feedback
  enable_touchscreen_manager: "false"  # Not on Matrix Voice
```

## Usage Examples

### Control LED Ring

```yaml
# Turn on LED ring with custom color
- service: light.turn_on
  target:
    entity_id: light.led_ring
  data:
    rgb_color: [255, 0, 0]  # Red
    brightness: 255

# Pulse animation
- service: light.turn_on
  target:
    entity_id: light.led_ring
  data:
    effect: "Pulse"
    rgb_color: [0, 0, 255]  # Blue
```

### Adjust Speaker Volume

```yaml
# Set speaker volume to 60%
- service: volume_set
  target:
    entity_id: media_player.va_media_player
  data:
    volume_level: 0.6
```

## Integration with Voice Assistant

The Matrix Voice integrates with shared packages and managers:

- **media_player.yaml** (device-local): defines `va_media_player` and includes shared media-player handlers
- **voice_assistant.yaml** (device-local): imports shared voice-assistant package
- **devices/includes/media_player/common.yaml**: shared media globals and startup sound switch
- **devices/includes/voice_assistant/common.yaml**: shared voice-assistant lifecycle and controls
- **managers/audio.yaml**: sound playback orchestration
- **managers/voice.yaml**: wake-word lifecycle ownership
- **managers/led_ring.yaml**: LED state management via event routing
- **event_router.yaml**: central event bus and manager routing

## Troubleshooting

### FPGA Communication Issues

1. Verify SPI pins: CLK (GPIO32), MOSI (GPIO33), MISO (GPIO21), CS (GPIO23)
2. Check FPGA firmware is loaded on Matrix Voice
3. Verify SPI clock frequency: 8MHz
4. Check external_components is correctly pointing to gnumpi/esphome_matrixio

### Audio Not Working

1. Check FPGA communication first (see above)
2. Verify microphone is enabled (check matrixio component logs)
3. Verify speaker is enabled in media_player configuration
4. Check speaker volume is not set to 0%

### LED Ring Not Responding

1. Verify FPGA communication (see above)
2. Check LED ring is powered
3. Verify LED ring entity is responding in Home Assistant
4. Check managers/led_ring.yaml is enabled for this device

## References

- [gnumpi/esphome_matrixio](https://github.com/gnumpi/esphome_matrixio) - Custom FPGA component
- [Matrix Voice Documentation](https://matrix-io.github.io/matrix-documentation/matrix-voice/resources/overview/) - Hardware documentation
- [Root Project README](../../README.md) - Multi-device architecture overview

