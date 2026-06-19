# Modules Integration Guide

This guide explains how to integrate the modules system into your event_router.yaml
and how to properly wire voice commands, display pages, LED feedback, and audio.

## Step 1: Enable Modules in event_router.yaml

Add module packages to `event_router.yaml`:

```yaml
packages:
  # Core managers (existing)
  interaction_manager: !include managers/interaction.yaml
  audio_manager: !include managers/audio.yaml
  voice_manager: !include managers/voice.yaml
  led_ring_manager: !include managers/led_ring.yaml
  touchscreen_manager: !include managers/touchscreen.yaml
  display_manager: !include managers/display.yaml
  
  # Modules (new)
  module_timer: !include modules/timer/module.yaml
  # module_weather: !include modules/weather/module.yaml
  # module_grocery_list: !include modules/grocery_list/module.yaml
```

Modules are loaded in order with all other packages. They can freely access
shared globals and systems.

## Step 2: Wire Voice Commands

In your voice assistant's `on_end` handler (typically in the voice manager),
route recognized intents to module scripts:

```yaml
voice_assistant:
  on_end:
    - lambda: |-
        std::string intent = id(recognized_intent);
        std::string transcript = id(speech_text);
        
        // Route to modules based on recognized intent
        if (intent == "timer.set") {
          // Extract duration from transcript and call timer
          uint32_t duration_sec = extract_duration(transcript);
          id(mm_timer_command_set)(duration_sec);
        }
        else if (intent == "timer.cancel") {
          id(mm_timer_command_cancel)();
        }
```

Or use a more structured approach with a decision tree in event_router.yaml.

## Step 3: Wire Display Pages

In the display manager's `display_manager_handle_interaction_state` script,
add logic to render module pages:

```yaml
script:
  - id: display_manager_handle_interaction_state
    then:
      - script.execute: display_manager_recompute_view
      - if:
          condition:
            lambda: 'return id(dm_current_view).find("module_") == 0;'
          then:
            - script.execute: display_manager_render_module_page
```

Then implement `display_manager_render_module_page` to check the page string
and render accordingly:

```yaml
  - id: display_manager_render_module_page
    then:
      - if:
          condition:
            lambda: 'return id(dm_current_view) == "module_timer:countdown";'
          then:
            - script.execute: render_timer_countdown_page
      - if:
          condition:
            lambda: 'return id(dm_current_view) == "module_timer:expired";'
          then:
            - script.execute: render_timer_expired_page
```

## Step 4: LED and Audio

These typically work automatically:

- **LED**: Modules trigger `led_event` directly, which the LED manager handles
- **Audio**: Modules call `audio_queue_sound` script, which the audio manager handles

No additional wiring needed if these systems are already in place.

## Step 5: System Events (Optional)

If modules need to react to assistant lifecycle, they can listen to `system_event`.
This is optional and depends on module requirements.

## Complete Example: Timer Module Flow

```
1. User says: "Set a timer for 5 minutes"
         ↓
2. Voice Assistant recognizes intent → calls mm_timer_command_set(300)
         ↓
3. Timer module:
   - Sets mm_timer_duration_sec = 300
   - Sets mm_timer_is_active = true
   - Triggers led_event "processing"
   - Queues audio "timer_started"
   - Sets dm_current_view = "module_timer:countdown"
         ↓
4. Display manager detects "module_" prefix and renders timer page
         ↓
5. LED manager renders "processing" state animation
         ↓
6. Audio manager plays confirmation sound
         ↓
7. Every second: mm_timer_tick checks if elapsed >= duration
         ↓
8. After 300 seconds:
   - Sets mm_timer_is_expired = true
   - Sets dm_current_view = "module_timer:expired"
         ↓
9. Display shows "Timer Expired"
   LED shows "error" state
   Audio plays repeating alarm
         ↓
10. User says "Stop" → calls mm_timer_command_acknowledge_expired()
         ↓
11. Timer module:
    - Sets mm_timer_is_expired = false
    - Calls audio_stop_sound
    - Resets dm_current_view to idle
    - Triggers led_event "idle"
         ↓
12. System returns to idle state
```

## Testing Modules

To test a module:

1. Include it in event_router.yaml
2. Compile and flash: `esphome compile base.yaml && esphome upload base.yaml`
3. Check logs for module startup and command execution
4. Test voice commands or directly call scripts via ESPHome dev tools

Example direct script call (for testing without voice):
```
Home Assistant → Developer Tools → Services → ESPHome
Service: esphome.mm_timer_command_set
Parameters: {"duration_sec": 300}
```

## Module Checklist

Before publishing a module:

- [ ] module.yaml with substitutions, globals, scripts
- [ ] README.md explaining features and usage
- [ ] VOICE_COMMANDS.md documenting all intents
- [ ] DISPLAY_PAGES.md documenting display views
- [ ] LED_AND_SOUNDS.md documenting audio/LED integration
- [ ] Naming conventions followed (mm_{name}_ prefix)
- [ ] State variables properly isolated
- [ ] No external dependencies (except core system)
- [ ] Tested and compiles without errors
- [ ] Documentation examples included

## Troubleshooting

### Module doesn't load
- Check syntax in module.yaml
- Ensure file path is correct in event_router.yaml
- Check for ID collisions with other modules

### Voice commands not recognized
- Verify intent routing in voice_assistant.on_end
- Check script IDs match what voice assistant is calling
- Test script manually via Dev Tools

### Display page not rendering
- Check if dm_current_view is being set correctly (log it)
- Verify display manager has logic to handle "module_" prefix
- Check page key spelling matches display logic

### LED/Audio not working
- Verify led_event and audio_queue_sound scripts exist
- Check sound keys are registered in audio system
- Verify LED manager handles custom event types

## Best Practices

1. **Isolation**: Keep all state and logic in the module
2. **Naming**: Use mm_{name}_ prefix consistently
3. **Documentation**: Document all voice commands and display pages
4. **Error Handling**: Log errors and gracefully handle edge cases
5. **Testing**: Test in isolation before deploying with other modules
6. **Reusability**: Design modules to work with multiple devices
7. **Performance**: Avoid expensive operations in 1s intervals
8. **State**: Use restore_value: no for all module globals (stateless)
