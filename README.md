# ESPHome Voice Assistant

A modular ESPHome configuration framework for voice assistant devices. Currently supports the **EchoEar** device (ESP32-S3 with 360×360 round touchscreen, gesture detection, battery monitoring, and audio I/O). Architecture designed for multi-device support.

## Attribution

This project is based on the original work by **RealDeco** at [xiaozhi-esphome](https://github.com/RealDeco/xiaozhi-esphome/). The original monolithic configuration has been refactored into a modular architecture following separation of concerns principles.

## About This Project

This project provides a production-ready, modular configuration for building voice assistant devices with ESPHome. It emphasizes:

- **Separation of Concerns**: Hardware, I/O, features, and shared logic are clearly separated
- **Event-Driven Architecture**: Centralized event dispatchers decouple audio, gesture, and display systems
- **Multi-Device Support**: Device-specific code is isolated; new hardware variants inherit shared functionality
- **Internationalisation**: All user-facing strings (gestures, labels, status messages) are externalized for translation
- **Voice Assistant Integration**: On-device wake word detection or Home Assistant voice assistant with dual codec audio support

## Quick Start

1. Clone the repository
2. Update `secrets.yaml` with WiFi credentials
3. Run: `esphome compile base.yaml && esphome flash base.yaml`

For detailed setup, see [EchoEar Documentation](devices/espressif/echoear/README.md).

## Supported Devices

### EchoEar (ESP32-S3)

A round-screen voice assistant with gesture control and battery monitoring.

- **Display**: 360×360 MIPI SPI touchscreen
- **Audio**: Microphone (I2S) and speaker (I2S) with dual codec support
- **Battery**: I2C fuel gauge (BQ27220)
- **Touch**: CST816 resistive + ESP32 capacitive pads
- **Status**: Green LED, adjustable backlight

[📖 EchoEar Documentation](devices/espressif/echoear/README.md)

## Adding Device Support

To add a new device (e.g., ESP32-C3):

1. Create folder: `devices/espressif/yourdevice/`
2. Create hardware configuration files (see EchoEar for template)
3. Update `base.yaml` substitution: `device: espressif/yourdevice`
4. See [Supported Devices](#supported-devices) for device-specific docs

## Architecture

The configuration uses a **tiered modular architecture** that separates device-specific code from shared functionality:

### Shared Modules (Root Level)

- **`base.yaml`** - Main entry point, orchestrates device and shared modules
- **`config.yaml`** - Global substitutions and configuration (sound URLs, etc.)
- **`event_router.yaml`** - Central event platform and routing to state managers
- **`i18n.yaml`** - Translatable strings (gesture names, entity labels, voice assistant settings)

### Device Modules (devices/ Folder)

Device-specific modules and shared device functionality:

- **`devices/voice_assistant.yaml`** - Wake word detection (on-device or Home Assistant), voice assistant lifecycle
- **`devices/media_player.yaml`** - Speaker control, audio playback, announcement pipeline

### YAML Includes (Partial Fragments - devices/includes/)

Partial YAML files intended to be included into other modules:

- **`devices/includes/sounds.yaml`** - Audio file definitions (included as media_player.files array)
- **`devices/includes/touchscreen_on_touch.yaml`** - Touch initiation handler (included as touchscreen.on_touch action)
- **`devices/includes/touchscreen_on_update.yaml`** - Touch movement tracking handler (included as touchscreen.on_update action)
- **`devices/includes/touchscreen_on_release.yaml`** - Gesture classification handler (included as touchscreen.on_release action)
- **`devices/includes/media_player/on_announcement.yaml`** - Announcement event handler
- **`devices/includes/media_player/on_idle.yaml`** - Media player idle event handler

### Device-Specific Modules

Device configurations live in `devices/vendor/model/` folders:

```
devices/
  espressif/
    echoear/
      device.yaml                  # Orchestrates device modules
      hardware.yaml                # ESP32-S3 setup, I/O buses
      io_entities.yaml             # GPIO/audio abstractions
      battery.yaml                 # BQ27220 monitoring
      touchscreen.yaml             # CST816 driver, gesture detection
```

### Module Dependencies

```
base.yaml
    ↓
    ├─→ config.yaml (substitutions: sound URLs, device path, etc.)
    ├─→ event_router.yaml (event platform, state manager routing)
    ├─→ i18n.yaml (translatable strings)
    │
    ├─→ manager/
    │   ├─→ audio.yaml (audio state and coordination)
    │   ├─→ voice.yaml (wake word lifecycle)
    │   ├─→ led_ring.yaml (LED ring feedback)
    │   └─→ touchscreen.yaml (gesture state tracking)
    │
    ├─→ devices/voice_assistant.yaml
    │   (wake word detection, voice assistant lifecycle)
    │
    ├─→ devices/media_player.yaml
    │   (speaker control, audio playback)
    │   └─→ devices/includes/sounds.yaml (audio files)
    │   └─→ devices/includes/media_player/on_announcement.yaml
    │   └─→ devices/includes/media_player/on_idle.yaml
    │
    └─→ devices/espressif/echoear/device.yaml
        ↓
        ├─→ hardware.yaml (raw I/O buses)
        ├─→ io_entities.yaml (GPIO/audio abstractions)
        ├─→ battery.yaml (I2C sensor)
        └─→ touchscreen.yaml (driver + gesture detection)
            ├─ devices/includes/touchscreen_on_touch.yaml (touch initiation)
            ├─ devices/includes/touchscreen_on_update.yaml (movement tracking)
            └─ devices/includes/touchscreen_on_release.yaml (gesture classification)
```

### Module Responsibilities

- **config.yaml**: Global substitutions and configuration (sound URLs, device path, event type constants, etc.)
- **event_router.yaml**: Central event platform definition and routing to state managers
- **i18n.yaml**: Translatable strings (gesture names, entity labels, voice settings, status messages)
- **manager/audio.yaml**: Central audio event dispatcher, microphone/speaker state coordination
- **manager/voice.yaml**: Wake word lifecycle management and control
- **manager/led_ring.yaml**: LED ring state feedback
- **manager/touchscreen.yaml**: Gesture state tracking, event definitions, master control, and event routing
- **hardware.yaml**: Raw I/O buses only (I2C, I2S, SPI, codecs)
- **io_entities.yaml**: GPIO/audio abstractions (microphone, speaker, not hardware configuration)
- **battery.yaml**: Sensor polling and calibration (self-contained)
- **touchscreen.yaml**: Touchscreen driver and gesture state management (wires callbacks to globals)
- **device.yaml**: Bundles all device-specific modules
- **devices/voice_assistant.yaml**: Wake word detection (on-device or Home Assistant), voice assistant lifecycle, audio event dispatch
- **devices/media_player.yaml**: Speaker control, audio playback, announcement pipeline, audio event dispatch
- **devices/includes/sounds.yaml**: Audio file definitions (partial, included as media_player.files array)
- **devices/includes/touchscreen_on_touch.yaml**: Touch initiation handler (partial, included as touchscreen.on_touch action)
- **devices/includes/touchscreen_on_update.yaml**: Touch movement tracking handler (partial, included as touchscreen.on_update action)
- **devices/includes/touchscreen_on_release.yaml**: Gesture classification handler (partial, included as touchscreen.on_release action)

### Adding a New Device Variant

To add support for a new device (e.g., ESP32-C3):

1. Create device folder: `devices/espressif/newdevice/`
2. Add device-specific files:
   - `device.yaml` (orchestrator)
   - `hardware.yaml` (board config)
   - `io_entities.yaml` (GPIO/audio setup)
   - `battery.yaml` (sensor config, may be identical)
   - `touchscreen.yaml` (display driver, may be identical)
3. Update `base.yaml` to reference new device path
4. No changes needed to shared modules: `devices/includes/`, `manager/`, `devices/voice_assistant.yaml`, or `devices/media_player.yaml`—they're device-agnostic

## Gesture System

The framework provides an event-driven gesture detection system with **vendor-neutral touch classification** that works across device variants:

- **Event-Driven Architecture**: Gesture detection triggers events routed to display handlers
- **Shared Gesture Logic**: Three vendor-neutral handlers (`devices/includes/touchscreen_on_touch.yaml`, `devices/includes/touchscreen_on_update.yaml`, `devices/includes/touchscreen_on_release.yaml`) provide reusable, hardware-agnostic gesture detection
- **Shared Event Routing**: `manager/touchscreen.yaml` handles all touchscreen events and state
- **Device-Specific Drivers**: Each device's `touchscreen.yaml` handles hardware driver + simple touch callback wiring

### Gesture Detection Design

**Vendor-neutral gesture classification** (`devices/includes/touchscreen_on_touch.yaml`, `devices/includes/touchscreen_on_update.yaml`, `devices/includes/touchscreen_on_release.yaml`):

- `on_touch` initializes gesture state (start position, timestamp)
- `on_update` tracks movement during active touch
- `on_release` classifies touch coordinates and timing into 7 gesture types
- Works with **any touchscreen driver** that provides `touch.x`, `touch.y`, and duration
- Parameterized via substitutions (rotation, thresholds, event names)
- Outputs touchscreen events that are routed by `manager/touchscreen.yaml` event dispatcher

**Device integration** (e.g., `devices/espressif/echoear/touchscreen.yaml`):

```yaml
touchscreen:
  - platform: cst816                    # CST816-specific driver
    on_touch:
      !include ../../includes/touchscreen_on_touch.yaml

    on_update:
      !include ../../includes/touchscreen_on_update.yaml

    on_release:
      !include ../../includes/touchscreen_on_release.yaml  # Vendor-neutral classifier
```

This design means **new touchscreen devices only need to**:
1. Wire `on_touch`, `on_update`, `on_release` to populate global state
2. Ensure `manager/touchscreen.yaml` handles all touchscreen events and state (shared across devices)

### Gesture Types

- `"Swipe Left"` - Horizontal swipe left
- `"Swipe Right"` - Horizontal swipe right
- `"Swipe Up"` - Vertical swipe up
- `"Swipe Down"` - Vertical swipe down
- `"Single Press"` - Quick tap
- `"Long Press"` - 500ms+ hold
- `"Double Press"` - Two taps within 400ms

### Gesture Timing Configuration

Gesture timing thresholds and display rotation are configured at the project level in `base.yaml` and apply to all devices:

```yaml
substitutions:
  rotate_display: "0"             # Display rotation (0/90/180/270) - adjust for your mounting angle
  long_press_duration: "500"      # Hold time threshold (ms)
  double_press_window: "400"      # Double-tap window (ms)
```

For device-specific gesture detection and battery monitoring implementation, see [EchoEar Gesture Detection](devices/espressif/echoear/README.md#gesture-detection) and [EchoEar Battery Monitoring](devices/espressif/echoear/README.md#battery-monitoring).

## Media Player

The framework includes audio playback capabilities for announcements, startup sounds, and audio feedback.

- **Media Player**: Speaker platform with FLAC announcement pipeline (48kHz mono)
- **Sound Files**: Wake word, timer finished, startup sound
- **Startup Sound**: Plays automatically on API client connection if enabled
- **Volume Control**: Configurable limits (0.5-0.7 range default)

For device-specific media player configuration (speaker control, sound files), see [EchoEar Media Player](devices/espressif/echoear/README.md#media-player).

## Sound Configuration

All audio file URLs are centralized in `config.yaml` for easy customization:

```yaml
substitutions:
  xiaozhi_sounds: "https://github.com/RealDeco/xiaozhi-esphome/raw/main/sounds"
  sound_startup: "${xiaozhi_sounds}/Home_Connected.flac"
  sound_wake_word: "${xiaozhi_sounds}/wake_word_triggered.flac"
  sound_timer_finished: "${xiaozhi_sounds}/timer_finished.flac"
```

Audio files are defined in `devices/includes/sounds.yaml` and included into the media_player configuration. To use custom sounds:

1. Host audio files (FLAC format recommended, 48kHz mono)
2. Edit `config.yaml` with new URLs
3. Or override `xiaozhi_sounds` to point to your own sound directory

## Voice Assistant System

The framework provides a fully **event-driven voice assistant architecture** with strict separation of concerns. Wake word lifecycle, audio playback, and I2S microphone capture are each owned by a dedicated manager, communicating only through a central event bus.

### Architecture Overview

```
devices/voice_assistant.yaml   ──publishes events──→  system_event (event bus)
devices/media_player.yaml      ──publishes events──→  system_event
va_microphone switch           ──publishes events──→  system_event
api on_connect/disconnect      ──publishes events──→  system_event
                                                           │
                                          event_router.yaml routes to:
                                                           │
                         ┌─────────────────────────────────┤
                         ↓                                 ↓
               manager/audio.yaml              manager/voice.yaml
             (sound playback only)        (wake word lifecycle owner)
```

**Key Principle**: No manager calls another manager's scripts directly. All coordination happens via events.

### Event Types

| Event | Published by | Meaning |
|---|---|---|
| `api_connected` | api component | Connected to Home Assistant |
| `api_disconnected` | api component | Disconnected from Home Assistant |
| `audio_ready` | audio manager | Startup sound flow complete; safe to arm wake word |
| `announcement_started` | media_player | External HA announcement playback began |
| `media_player_idle` | media_player | Announcement/sound playback finished |
| `mic_muted` | va_microphone switch | Mic switch turned off (capture stopped) |
| `mic_unmuted` | va_microphone switch | Mic switch turned on |
| `wake_word_stop_requested` | any component needing mic free | Signals Voice Manager to stop wake word |
| `wake_word_location_changed` | wake word location select | Device/HA mode switched |
| `voice_assist_detected` | micro_wake_word | Wake word detected |
| `voice_assist_listening` | voice_assistant | Microphone listening for speech |
| `voice_assist_thinking` | voice_assistant | STT processing |
| `voice_assist_replying` | voice_assistant | TTS started |
| `voice_assist_idle` | voice_assistant | Session ended (on_end fired) |
| `voice_assist_error` | voice_assistant | Error occurred |

### Manager Responsibilities

**`manager/voice.yaml`** — sole owner of wake word lifecycle:
- Decides when to start/stop `micro_wake_word` and `voice_assistant`
- Gate conditions: mic switch ON + API connected + VA not running
- Starts `va_microphone` switch as part of `start_wake_word`
- Responds to: `audio_ready`, `media_player_idle`, `mic_unmuted`, `va_idle`, `va_detected`, `api_disconnected`, `announcement_started`, `mic_muted`, `wake_word_stop_requested`

**`manager/audio.yaml`** — sound playback and audio feedback:
- Plays wake sound on `va_detected` (if enabled)
- On `api_connected`: emits `wake_word_stop_requested`, waits for mic idle, plays startup sound, then emits `audio_ready`
- Never calls `stop_wake_word` directly

**`devices/voice_assistant.yaml`** — lifecycle events only:
- `on_wake_word_detected`: emits `wake_word_stop_requested`, waits for mic idle, plays optional beep, calls `voice_assistant.start`
- `on_listening/stt_vad_end/tts_start/end/error`: publish phase events to event bus
- `va_microphone` switch: `on_turn_off` calls `microphone.stop_capture` then emits `mic_muted`; `on_turn_on` emits `mic_unmuted` only (Voice Manager owns capture re-arm)

**`devices/media_player.yaml`** — announcement signaling:
- `on_announcement`: emits `announcement_started` for external HA announcements only (internal sounds from `play_sound` are event-silent to avoid recursion)
- `on_idle`: emits `media_player_idle` for external announcements only

### Boot Startup Flow

```
boot
  → priority:800 on_boot: stop VA + micro_wake_word
  → HA API connects
  → audio_manager: emit wake_word_stop_requested
  → audio_manager: wait for mic idle
  → audio_manager: play startup sound
  → audio_manager: emit audio_ready
  → voice_manager: start_wake_word (gate check passes)
  → micro_wake_word.start (device mode) or voice_assistant.start_continuous (HA mode)
```

### Wake Word Detection Modes

Configure in Home Assistant UI (Wake word engine location):

1. **On-Device Mode** (default):
   - Local detection using `micro_wake_word` component
   - Models: `okay_nabu`, `hey_jarvis`, `alexa`
   - No internet dependency, lower latency

2. **Home Assistant Mode**:
   - Detection via Home Assistant Assist pipeline
   - Uses `voice_assistant.start_continuous`
   - Full HA voice model selection

### Microphone & Speaker Controls

**User Controls** (Home Assistant UI):
- **Wake Sound**: Beep when wake word detected (default: ON)
- **Startup Sound**: Beep on boot/API connect (default: ON)
- **Microphone**: Toggle microphone on/off (stops I2S capture when off)
- **Wake word engine location**: Switch between on-device and HA wake word

### Adding to New Devices

When adding a new device:
- Configure I2S microphone as `box_mic` in `io_entities.yaml`
- Configure I2S speaker as `box_speaker` in `io_entities.yaml`
- Load audio codec drivers in `hardware.yaml`
- No changes needed to `devices/voice_assistant.yaml`, `devices/media_player.yaml`, `manager/audio.yaml`, or `manager/voice.yaml`

## Internationalisation (i18n)

All user-facing strings (gesture names, entity labels, status messages) are externalized for easy translation.

### i18n Files

**Root level** (`i18n.yaml`):
- Touchscreen event names (Swipe Left, Long Press, etc.)
- Generic labels (Microphone, Speaker, Battery Voltage, etc.)
- Status messages (Charging, Discharging, Full, Empty)
- Device description

**Device level** (`devices/espressif/echoear/i18n.yaml`):
- Device model name (EchoEar)
- Device-specific labels (Back Touch Pad, Head Touch Pad, Green LED)

### Translating to Another Language

1. Edit `i18n.yaml` (root) for generic strings:
   ```yaml
   substitutions:
     touchscreen_swipe_left: "Deslizar izquierda"
     touchscreen_long_press: "Presión larga"
     label_battery_voltage: "Voltaje de batería"
     device_description: "Dispositivo Asistente de Voz"
   ```

2. Edit `devices/espressif/echoear/i18n.yaml` for device-specific strings:
   ```yaml
   substitutions:
     device_model: "EchoEar"
     label_back_touch_pad: "Almohadilla táctil trasera"
     label_head_touch_pad: "Almohadilla táctil delantera"
     label_green_led: "LED verde"
   ```

3. Strings are automatically applied across all files that reference them

### Adding Translations for New Devices

When adding a new device variant (e.g., ESP32-C3), create `devices/espressif/newdevice/i18n.yaml` with only device-specific translations. All generic strings are inherited from root `i18n.yaml`.

## Configuration

### Initial Setup

1. Clone the repository
2. Update `secrets.yaml` with WiFi credentials:
   ```yaml
   wifi_ssid: "Your SSID"
   wifi_password: "Your Password"
   ```
3. Configure ESPHome with device hostname
4. Flash to ESP32-S3

### Building & Flashing

Requires **ESPHome 2025.9.0+** for MIPI SPI display driver support.

```bash
esphome compile base.yaml
esphome flash base.yaml
esphome logs base.yaml
```

### Device Customization

For device-specific customization (battery calibration, touch sensitivity, gesture detection), see:

- **[EchoEar Customization](devices/espressif/echoear/README.md#customization)**

For global settings (display rotation, gesture timing), edit `base.yaml` substitutions.

## Troubleshooting

**General Issues**:
- Verify ESPHome version is 2025.9.0+
- Check that device substitution in `base.yaml` matches your device folder name

**Device-Specific Issues**:

For EchoEar troubleshooting (gesture detection, battery monitoring, display rotation, etc.), see:
- **[EchoEar Troubleshooting](devices/espressif/echoear/README.md#troubleshooting)**

## Future Work

### Completed (2026-Q2)
- ✅ Event-driven voice assistant architecture with on-device and Home Assistant modes
- ✅ Central audio state management (manager/audio.yaml)
- ✅ Audio playback (startup sound, wake word beep, TTS response)
- ✅ Microphone mute/unmute control
- ✅ Gesture detection system (vendor-neutral)

### Pending
- **Display rendering**: Clock display, UI widgets, visual feedback animations
- **Timer integration**: Timer start/stop/finish with audio/visual feedback
- **Home Assistant entity mapping**: Advanced automation support and state exposure
- **Additional wake word models**: Custom model support beyond okay_nabu/hey_jarvis/alexa
- **Multi-language voice support**: Support for non-English voice assistants

---

**Status**: Production-ready for voice assistant (on-device and Home Assistant modes), gesture detection, and battery monitoring. Display rendering and advanced timer integration still in development.
