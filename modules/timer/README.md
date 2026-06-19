# Timer Module

A self-contained timer module that provides voice-controlled countdown functionality.

## Features

- **Voice Commands**: "Set a timer for X minutes/seconds", "Cancel the timer", "Stop" (to acknowledge expiry)
- **Display**: Shows countdown timer with remaining time, and an "expired" screen when finished
- **LED Feedback**: Processing state while active, error/alert state when expired
- **Sound**: Confirmation sound on start, repeating alarm on expiry
- **State Management**: Self-contained state (no external dependencies except for audio/LED/display infrastructure)

## Files

- `module.yaml` - Main timer logic, state, and scripts
- `VOICE_COMMANDS.md` - Voice command documentation
- `DISPLAY_PAGES.md` - Display page definitions
- `LED_AND_SOUNDS.md` - LED and audio integration
- `README.md` - This file

## How to Enable

Add to `event_router.yaml` in the packages section:

```yaml
packages:
  # ... other packages ...
  module_timer: !include modules/timer/module.yaml
```

## Integration Requirements

The timer module assumes these exist in the base system:

1. **Audio System**: `audio_queue_sound` script that accepts:
   - `sound_key`: "timer_started", "timer_alarm"
   - Optional: `audio_stop_sound` script to stop playback

2. **LED System**: `led_event` event that accepts:
   - `event_type`: "processing", "error", "idle"

3. **Display System**: Uses `dm_current_view` global to set display:
   - "module_timer:countdown" - Shows timer counting down
   - "module_timer:expired" - Shows timer expired screen

## State Variables

All prefixed with `mm_timer_`:

- `duration_sec` (uint32_t) - Total duration in seconds
- `started_ms` (uint32_t) - Millis when timer started
- `is_active` (bool) - Whether timer is counting down
- `is_expired` (bool) - Whether timer finished and awaiting acknowledgement

## Scripts

- `mm_timer_command_set(duration_sec)` - Start a new timer
- `mm_timer_command_cancel()` - Cancel active timer
- `mm_timer_command_acknowledge_expired()` - Acknowledge expired state
- `mm_timer_tick()` - Called every second (internal)

## Voice Integration

Wire these intents from your voice assistant to the timer scripts:

```yaml
# When voice assistant recognizes a set_timer intent:
script.execute:
  id: mm_timer_command_set
  duration_sec: <extracted duration>

# When voice assistant recognizes cancel_timer intent:
script.execute:
  id: mm_timer_command_cancel

# When in expired state and hears "stop" or "acknowledge":
script.execute:
  id: mm_timer_command_acknowledge_expired
```

## Example: Setting a Timer

1. User says: "Set a timer for 5 minutes"
2. Voice assistant recognizes intent and calls `mm_timer_command_set` with `duration_sec: 300`
3. Timer:
   - Stores 300 seconds as duration
   - Records current time
   - Sets `is_active = true`
   - Triggers LED "processing" state
   - Queues "timer_started" sound
   - Updates display to countdown view
4. Every second, `mm_timer_tick` checks elapsed time
5. After 300 seconds, sets `is_expired = true`
6. Display changes to "expired" screen
7. LED changes to "error" state
8. "timer_alarm" sound plays and repeats
9. User says "Stop" (or clicks screen)
10. `mm_timer_command_acknowledge_expired` is called
11. Returns to idle state

## Customization

To add sounds or change behavior:
- Edit the sound keys in `module.yaml` to match your audio system's keys
- Modify the LED event types to match your device's LED states
- Adjust timeout/display logic in the scripts as needed

All changes are isolated to this module directory.
