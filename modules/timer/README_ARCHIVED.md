# Timer Module - ARCHIVED

**Status: ARCHIVED** - All timer functionality is now part of core voice_assistant implementation.

## What Happened

Timers are **native to ESPHome's voice_assistant component**—they're not something we should build ourselves. We discovered this after initial development and refactored to move all timer functionality into core.

## Where Timers Live Now

All timer core functionality is now integrated directly into:

- **Core Implementation**: `devices/includes/voice_assistant/common.yaml`
  - Timer handler setup
  - Timer display rendering scripts
  - Timer state tracking (globals)
  - Home Assistant entity integration
  - LED/audio feedback integration

## Using Timers

Timers work automatically when you set up a voice_assistant component. No special module or configuration needed:

```yaml
voice_assistant:
  id: va
  microphone: va_microphone
  media_player: va_media_player
  
  # Timer handlers are built-in - they automatically work
  on_timer_started: [... handlers in common.yaml ...]
  on_timer_finished: [... handlers in common.yaml ...]
  on_timer_cancelled: [... handlers in common.yaml ...]
  on_timer_tick: [... handlers in common.yaml ...]
```

Voice users can say:
- "Set a timer for 5 minutes"
- "Cancel the timer"
- "Pause the timer"
- "Resume the timer"
- And HA will handle the intent routing

## Why This Module is Archived

Timers are **too fundamental** to be optional. They're core to any voice assistant device. Keeping them as a module meant:
1. ❌ Duplicated configuration complexity
2. ❌ Non-obvious dependency (where's timer support?)
3. ❌ Fighting against ESPHome's native timer implementation

Moving to core means:
1. ✅ Timers always work out of the box
2. ✅ Clear, obvious design (timers = voice assistant feature)
3. ✅ Works with ESPHome's native Timer struct
4. ✅ Less code duplication

## Historical Reference

This module directory is kept for reference. The implementation demonstrated good practice for timer state management before we realized ESPHome handles it natively.
