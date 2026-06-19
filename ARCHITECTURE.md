# Technical Architecture and Device Extension Guide

This document describes the internal architecture of the ESPHome Voice Assistant project and provides a practical guide for adding new hardware variants.

## Scope

Use this document for:

- Understanding module boundaries and dependency flow
- Understanding event-driven coordination between managers and device wrappers
- Adding a new device profile under `devices/<vendor>/<model>/`
- Extending the system with new feature managers

For setup and day-to-day usage, see the main project README.

## Architecture Overview

The project uses a tiered modular design:

- Root-level shared modules provide global config, routing, and feature managers.
- Device folders provide hardware-specific definitions and wrappers.
- Include fragments in `devices/includes/` provide reusable behavior blocks.

### High-Level Dependency Flow

```text
base.yaml
  -> config.yaml
  -> i18n.yaml
  -> devices/<vendor>/<model>/device.yaml
  -> event_router.yaml
     -> managers/
        -> interaction.yaml
        -> audio.yaml
        -> voice.yaml
        -> led_ring.yaml
        -> touchscreen.yaml
        -> display.yaml
```

### Device-Level Flow (EchoEar example)

```text
devices/espressif/echoear/device.yaml
  -> i18n.yaml
  -> hardware.yaml
  -> io_entities.yaml
  -> battery.yaml
  -> display.yaml
   -> ui.yaml
  -> touchscreen.yaml
     -> ../../includes/touchscreen/on_touch.yaml
     -> ../../includes/touchscreen/on_update.yaml
     -> ../../includes/touchscreen/on_release.yaml
  -> media_player.yaml
     -> ../../includes/media_player/common.yaml
     -> ../../includes/media_player/on_announcement.yaml
     -> ../../includes/media_player/on_idle.yaml
     -> ../../includes/sounds.yaml
  -> voice_assistant.yaml
     -> ../../includes/voice_assistant/common.yaml
```

## Core Modules

### Root Modules

- `base.yaml`: Entry point and load orchestration
- `config.yaml`: Global substitutions (device path, sound URLs, etc.)
- `event_router.yaml`: Central event routing to state managers
- `i18n.yaml`: Shared translatable strings

### State Managers

- `managers/interaction.yaml`: Canonical interaction-state owner (maps raw system events to `not_connected`, `idle`, `muted`, `listening`, `thinking`, `replying`, `error`) and publishes `interaction_state_changed`
- `managers/audio.yaml`: Audio event handling and startup sound coordination
- `managers/voice.yaml`: Wake-word lifecycle ownership and gating
- `managers/led_ring.yaml`: LED ring translation layer from canonical interaction state to LED events and device rendering
- `managers/touchscreen.yaml`: Gesture event/state tracking and gesture event dispatch
- `managers/display.yaml`: Display state/view model and render triggers; consumes canonical interaction state and touchscreen events

### Device Modules

Each device should define:

- `device.yaml`: Device orchestrator package
- `hardware.yaml`: Raw buses, codecs, board-level platform setup
- `io_entities.yaml`: Higher-level entities and IDs (microphone, speaker, GPIO abstractions)
- Optional device capabilities (for example `battery.yaml`, `touchscreen.yaml`)
- Optional UI capabilities (for example `display.yaml`, `ui.yaml`)
- Device wrappers: `media_player.yaml`, `voice_assistant.yaml`

### Shared Include Fragments

Common reusable fragments in `devices/includes/`:

- `voice_assistant/common.yaml`
- `media_player/common.yaml`
- `media_player/on_announcement.yaml`
- `media_player/on_idle.yaml`
- `sounds.yaml`
- `touchscreen/on_touch.yaml`
- `touchscreen/on_update.yaml`
- `touchscreen/on_release.yaml`

## Event-Driven Design

Most cross-module coordination is done through `system_event` routing into a canonical interaction-state layer.

### Principle

- Producers publish lifecycle events to `system_event`.
- `event_router.yaml` routes those raw events to managers.
- `managers/interaction.yaml` is the single semantic owner that maps raw lifecycle events to one canonical interaction state.
- Consumers (`managers/led_ring.yaml`, `managers/display.yaml`) react to `interaction_state_changed` instead of remapping raw events independently.
- Touch input remains a separate path: `touchscreen_event` -> `managers/touchscreen.yaml` -> `display_manager_handle_touch_event`.

### Canonical Interaction Flow

```text
voice/media/api/mic events
   -> system_event
   -> event_router.yaml
   -> managers/interaction.yaml (single mapping owner)
   -> interaction_state_changed
       -> managers/led_ring.yaml (state -> LED behavior)
       -> managers/display.yaml (state -> view model)
```

### Typical Event Sources

- Voice assistant lifecycle callbacks
- Media player announcement/idle callbacks
- API connect/disconnect callbacks
- Microphone control switch transitions

### Typical Event Consumers

- Audio manager for sound/state synchronization
- Voice manager for wake-word lifecycle transitions
- Interaction manager for canonical interaction semantics
- LED and display managers for feedback rendering from canonical state
- Touchscreen manager for input dispatch

## Voice Assistant Architecture

The voice pipeline is split by responsibility:

- Voice manager controls when wake-word engines and voice assistant can run.
- Audio manager handles startup sound and audio readiness synchronization.
- Device voice/media wrappers publish lifecycle events and include shared behavior.

### Boot Coordination Pattern

1. Device boots and voice engines are held/reset.
2. API connects.
3. Audio manager requests wake-word stop, waits for safe audio state, plays startup sound.
4. Audio manager publishes readiness event.
5. Voice manager re-arms wake-word flow once gates pass.

### Wake-Word Modes

- On-device wake-word mode (micro_wake_word)
- Home Assistant continuous mode (voice_assistant.start_continuous)

Mode selection is exposed through user controls and routed through event/state logic.

## Gesture Architecture

Gesture detection is designed to be vendor-neutral:

- Device driver reports touch events and coordinates.
- Shared include handlers classify gestures based on movement and timing.
- Touchscreen manager emits/dispatches resulting gesture events.
- Display manager owns page/detail navigation policy from those gesture events.

This allows reuse of classification logic across different touchscreen controllers.

## Internationalization Model

- Root `i18n.yaml` defines shared labels and gesture/status strings.
- Device-level `i18n.yaml` files define only device-specific labels.
- Device behavior overrides belong in device composition (`device.yaml`), not in i18n files.

The substitution strategy keeps localization centralized and avoids string duplication.

## Add a New Device

This checklist describes the expected minimum integration path.

### 1. Create Device Folder

Create:

- `devices/<vendor>/<model>/device.yaml`
- `devices/<vendor>/<model>/hardware.yaml`
- `devices/<vendor>/<model>/io_entities.yaml`
- Optional modules needed by the hardware/UI (for example `battery.yaml`, `touchscreen.yaml`, `display.yaml`, `ui.yaml`)
- `devices/<vendor>/<model>/media_player.yaml`
- `devices/<vendor>/<model>/voice_assistant.yaml`
- `devices/<vendor>/<model>/i18n.yaml`

### 2. Wire Device Orchestrator

In `device.yaml`, include modules in dependency-safe order:

1. `i18n.yaml`
2. `hardware.yaml`
3. `io_entities.yaml`
4. Peripheral/UI modules (`battery.yaml`, `display.yaml`, `ui.yaml`, `touchscreen.yaml`, etc.)
5. `media_player.yaml`
6. `voice_assistant.yaml`

### 3. Reuse Shared Includes

In device `media_player.yaml` and `voice_assistant.yaml`, include common behavior packages from `devices/includes/` rather than duplicating logic.

### 4. Provide Required Entity IDs

Ensure expected IDs used by shared packages exist and match semantics:

- Microphone entity (for voice assistant capture)
- Speaker/media player entities
- Any switch/select entities referenced by shared behavior

### 5. Configure Shared Substitutions

Set required substitutions in `base.yaml` and `config.yaml` (for example `device`, sound URLs, gesture timing, and display rotation).

### 6. Activate Device

Update `base.yaml` substitution:

```yaml
substitutions:
  device: <vendor>/<model>
```

### 7. Validate Build and Runtime

- `esphome compile base.yaml`
- `esphome flash base.yaml`
- `esphome logs base.yaml`

Check:

- API connect/disconnect transitions
- Wake-word arming/disarming behavior
- Media announcement behavior and idle transitions
- Touch/gesture behavior (if applicable)
- Device-specific sensors and controls

## Add a New Manager

To add a new subsystem manager while preserving architecture style:

1. Create `managers/<feature>.yaml` with isolated state and one event entrypoint.
2. Define event constants and payload schema via substitutions.
3. Add routing in `event_router.yaml` by mapping relevant `system_event` types to the new manager script.
4. Avoid direct coupling to other managers; communicate via events.
5. Document new events and ownership in this file.

## Guardrails and Conventions

- Prefer event constants/substitutions over hardcoded event strings.
- Keep one canonical owner for cross-cutting interaction semantics (`managers/interaction.yaml`).
- Keep state ownership local to the manager that owns behavior.
- Keep hardware specifics in device files, not shared managers.
- Keep includes focused and reusable (single concern per include file).
- Keep load order explicit and deterministic in orchestrator files.
- Keep router event lists and manager contracts synchronized when introducing new events.
- Keep UI render loops side-effect free; avoid triggering non-UI behavior from polling intervals.

## Related Documentation

- Main setup guide: `README.md`
- Device-specific details: `devices/espressif/echoear/README.md`
