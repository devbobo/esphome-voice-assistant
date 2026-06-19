################################################################################
## Modules - Extensible Functionality System
##
## This directory contains self-contained modules that extend the voice
## assistant with new capabilities. Each module can provide:
##   - State management (timers, counters, etc.)
##   - Voice commands and intents
##   - Display pages and UI
##   - LED feedback and animations
##   - Sounds and audio queues
##   - Custom event handling
##
## Modules are activated by including them in event_router.yaml packages.
################################################################################

# Module System Architecture

## How Modules Work

1. **Discovery**: Each module is a self-contained YAML package in `modules/{name}/`
2. **Activation**: Include the module in `event_router.yaml`:
   ```yaml
   packages:
     module_timer: !include modules/timer/module.yaml
     module_weather: !include modules/weather/module.yaml
   ```
3. **Integration**: Modules hook into:
   - Voice commands (via script calls from voice assistant)
   - Display system (by setting `dm_current_view`)
   - LED system (by triggering `led_event`)
   - Audio system (by calling `audio_queue_sound`)
4. **Isolation**: Each module owns its own state and doesn't interfere with others

## Module Structure

Each module directory contains:

```
modules/{module_name}/
  module.yaml                 - Main logic (substitutions, globals, scripts, intervals)
  README.md                   - Overview and usage guide
  VOICE_COMMANDS.md          - Voice command documentation
  DISPLAY_PAGES.md           - Display page definitions
  LED_AND_SOUNDS.md          - LED and audio integration notes
  handlers/                   - Complex logic files (optional)
```

## Naming Conventions

All module-owned items use a `mm_{module_name}_` prefix:

- State variables: `mm_timer_duration_sec`, `mm_weather_current_temp`
- Scripts: `mm_timer_command_set`, `mm_weather_update`
- Display pages: `module_timer:countdown`, `module_weather:forecast`
- Sounds: `timer_alarm`, `weather_alert` (no prefix needed for sound keys)

This prevents collisions between modules and the core system.

## Core Integration Points

### 1. Voice Commands

Modules can implement scripts that are called when voice intents are recognized:

```yaml
script:
  - id: mm_timer_command_set
    parameters:
      duration_sec: uint32_t
    then:
      - lambda: id(mm_timer_duration_sec) = duration_sec;
```

The voice assistant's `on_end` handler routes intents to these scripts.

### 2. Display Pages

Modules set display views by updating `dm_current_view`:

```yaml
- lambda: id(dm_current_view) = "module_timer:countdown";
```

The display manager renders the appropriate page based on this string.

### 3. LED Feedback

Modules trigger LED events:

```yaml
- event.trigger:
    id: led_event
    event_type: "processing"  # or "listening", "idle", "error", etc.
```

The LED manager translates these to hardware animations.

### 4. Audio/Sounds

Modules queue sounds via the audio system:

```yaml
- script.execute: audio_queue_sound
  parameters:
    sound_key: "timer_started"
```

### 5. System Events (Optional)

Modules can listen to `system_event` for assistant lifecycle awareness:

```yaml
- if:
    condition:
      lambda: 'return event_type == "${evt_va_idle}";'
    then:
      - script.execute: mm_timer_handle_assistant_idle
```

## Example: Timer Module

The `timer/` directory contains a complete example:

- User says "set a timer for 5 minutes"
- Voice assistant calls `mm_timer_command_set(300)`
- Timer module:
  - Stores 300 seconds
  - Triggers LED "processing" state
  - Queues "timer_started" sound
  - Updates display to countdown view
- Every second, checks if time elapsed ≥ duration
- When expired:
  - Sets `is_expired = true`
  - Changes display to "expired" screen
  - Triggers LED "error" state
  - Queues repeating "timer_alarm" sound
- User says "stop" → acknowledges expiry → returns to idle

## Adding Your Own Module

1. Create `modules/{name}/` directory
2. Copy files from `modules/_template/`
3. Define your state in globals
4. Implement scripts for your commands
5. Document voice commands, display pages, and sounds
6. Include in `event_router.yaml`:
   ```yaml
   module_my_feature: !include modules/my_feature/module.yaml
   ```
7. Test and compile

## Module Dependencies

Modules depend on the core system providing:

- `dm_current_view` - Global for display page routing
- `led_event` - Event platform for LED feedback
- `audio_queue_sound` - Script for queueing sounds
- `voice_assistant` - Home Assistant voice assistant platform
- Optionally: `system_event` - For lifecycle awareness

These are all provided by the main system, so modules are portable.

## Composability

Modules are designed to be independent and composable:
- No module A → module B dependencies (unless explicitly designed)
- Each module manages its own state
- They coordinate via shared infrastructure (LED, display, audio)
- You can enable/disable modules by including/excluding them from event_router.yaml

## Future Ideas

Possible extensions:

- **Grocery List**: Voice-controlled todo list with display
- **Weather**: Voice-triggered weather updates with display/sounds
- **Reminders**: Voice-set reminders with notification sounds
- **Alarms**: Multiple alarms with custom wake sounds
- **Custom Routines**: Chained voice commands
- **Games**: Voice-controlled games with LED/display feedback
- **Music Playlist**: Voice control for music selection
- **Smart Home Scenes**: Voice-triggered device scenes
