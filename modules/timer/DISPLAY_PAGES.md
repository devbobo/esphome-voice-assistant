# Timer Module - Display Pages

This file documents the display pages provided by the timer module.

## Pages

### Timer Countdown: `module_timer:countdown`

Displays the current timer state with:
- Remaining time (MM:SS format)
- Progress bar or visual countdown
- Prompt to "Say stop or cancel to stop timer"

**View Variables Available**:
- `mm_timer_duration_sec` - Total duration
- `mm_timer_started_ms` - Start time
- Calculate remaining: `(duration - (millis - started) / 1000)`

### Timer Expired: `module_timer:expired`

Displays when timer has finished:
- "Timer Expired" message
- Asks user to "Say stop to acknowledge"
- Could show preset reminders ("Set another timer?")

**Triggered By**: `mm_timer_command_acknowledge_expired` script

## Implementation

To implement these pages in your device's display system, listen for these view keys
and render appropriately.

Example in display manager:
```yaml
- if:
    condition:
      lambda: 'return id(dm_current_view).find("module_timer:") == 0;'
    then:
      - script.execute: render_timer_page
```
