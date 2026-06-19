# Module Template

This is a template for creating new modules in the voice assistant system.

## Structure

Each module is self-contained and includes:

- `module.yaml` - Main module composition and lifecycle (required)
- `config.yaml` - Substitutions and constants (optional)
- `state.yaml` - Module state globals (optional)
- `voice_commands.yaml` - Voice command registration (optional)
- `display_pages.yaml` - UI page definitions (optional)
- `led_feedback.yaml` - LED state/event mapping (optional)
- `sounds.yaml` - Audio assets and sound queuing (optional)
- `handlers/` - Include files for complex logic (optional)

## How Modules Work

Modules are activated by including them in `event_router.yaml`:

```yaml
packages:
  module_my_module: !include modules/my_module/module.yaml
```

Each module can:
- Own its own state (via globals)
- Register voice commands
- Define display pages
- Map LED feedback
- Queue sounds
- Listen to system events (optionally)
- Emit custom events

## Module Integration Points

### 1. Voice Commands
Modules can define voice commands like:
```yaml
scripts:
  my_module_handle_command:
    mode: queued
    then:
      # Handle the command
```

These are called from `voice_assistant.on_end` after speech recognition.

### 2. Display Pages
Modules can define pages keyed by `module_name:page_key`:
```yaml
globals:
  - id: mm_current_page
    type: std::string
    initial_value: '"idle"'
```

### 3. LED Feedback
Modules can trigger LED events:
```yaml
- event.trigger:
    id: led_event
    event_type: "${led_event_listening}"
```

### 4. Sounds
Modules can queue sounds:
```yaml
- script.execute:
    id: audio_queue_sound
    sound_key: "timer_expired"
```

### 5. Timers & Intervals
Modules can use ESPHome intervals for background tasks:
```yaml
interval:
  - interval: 1s
    then:
      - script.execute: my_module_tick
```

## Naming Conventions

- Module folder: `lowercase_name`
- State/script IDs: `mm_{module_name}_{purpose}` (e.g., `mm_timer_elapsed_ms`)
- Voice commands: `mm_{module_name}_command_{command_name}`
- Display pages: `module_{module_name}:{page_key}`
- LED events: Use standard `led_event` with canonical states
- Sounds: `module_{module_name}_{sound_name}`

## Example

See `modules/timer/` for a complete example timer module.
