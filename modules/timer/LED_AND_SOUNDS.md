# Timer Module - LED and Sound Integration

## LED States

The timer module uses the standard LED event types:

- **Active Timer** (counting down): `processing`
  - Visual: Pulsing green or animation indicating activity

- **Expired** (waiting for acknowledgement): `error`
  - Visual: Solid red or blinking red alert

- **Idle** (timer cancelled or acknowledged): `idle`
  - Visual: Normal idle state

These are triggered directly via the `led_event` platform.

## Sounds

The timer module queues sounds via the `audio_queue_sound` script:

- **`timer_started`**: Confirmation beep/chime when timer is set
  - Duration: ~500ms
  - Type: Positive confirmation tone

- **`timer_alarm`**: Repeating alarm sound when timer expires
  - Duration: ~1-2 seconds, repeats every 1 second until acknowledged
  - Type: Alert/attention-getting tone
  - Loops until `audio_stop_sound` is called

## Integration

Both LED and sound events are triggered by the timer scripts automatically:
- When timer is set → `processing` LED + `timer_started` sound
- Every 1 second while expired → LED updates + `timer_alarm` sound queued
- When acknowledged → `idle` LED + sound stops

No additional wiring needed in display or LED managers.
