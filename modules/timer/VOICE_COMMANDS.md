# Timer Module - Voice Commands

This file documents the voice commands that the timer module responds to.
These should be configured in your voice assistant pipeline.

## Commands

### Set Timer
- "set a timer for {minutes} minutes"
- "set a timer for {seconds} seconds"
- "timer for {duration}"

**Handler**: `mm_timer_command_set` with `duration_sec` parameter

### Cancel Timer
- "cancel the timer"
- "stop the timer" (only works when timer is active, not expired)
- "clear the timer"

**Handler**: `mm_timer_command_cancel`

### Acknowledge Expired
- "stop" (when timer is expired)
- "acknowledge" (when timer is expired)

**Handler**: `mm_timer_command_acknowledge_expired`

## Implementation Notes

The voice commands are typically wired up in the voice assistant's `on_end` handler
to call the appropriate timer scripts based on recognized intents.

For example, in your voice assistant configuration:
```yaml
on_end:
  - if:
      condition:
        lambda: 'return recognized_intent == "set_timer";'
      then:
        - script.execute:
            id: mm_timer_command_set
            duration_sec: !lambda 'return extracted_duration;'
```
