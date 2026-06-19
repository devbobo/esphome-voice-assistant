################################################################################
## Voice Intent Routing Architecture
##
## How voice commands are routed to modules
################################################################################

## Current Limitation

ESPHome's VoiceAssistant component does not expose intent data in the `on_end` 
handler. The intent matching and routing must happen in Home Assistant, not on 
the device.

## Architecture

```
1. Device records voice → ESPHome VoiceAssistant
2. Audio sent to Home Assistant
3. Home Assistant runs STT (speech-to-text)
4. Home Assistant matches intent (timer_set_minutes, etc.)
5. Home Assistant calls device service/action
6. ESPHome script executes: mm_timer_voice_handler()
```

## Implementation

### On Device (ESPHome)

Each module implements a voice handler script:

```yaml
# modules/timer/voice.yaml
script:
  - id: mm_timer_voice_handler
    parameters:
      intent_name: string
      intent_data: string
    then:
      - if:
          condition: 
            lambda: 'return intent_name == "timer_set_minutes";'
          then:
            - script.execute:
                id: mm_timer_command_set
                duration_sec: !lambda 'return parse_minutes_from_data(intent_data);'
```

### In Home Assistant

Create automations or scripts to call device scripts when intents match:

```yaml
# Home Assistant automation
automation:
  - alias: "Echo Ear Timer - Set Minutes"
    trigger:
      platform: event
      event_type: intent_recognized
      event_data:
        intent_type: timer_set_minutes
    action:
      - service: esphome.echo_ear_mm_timer_voice_handler
        data:
          intent_name: "timer_set_minutes"
          intent_data: "{{ trigger.event.data }}"
```

Or use a custom component to map intents to device services.

## Future Enhancement

If ESPHome adds intent data to the `on_end` handler, we can implement in-device 
routing:

```yaml
# When available
on_end:
  - if:
      condition:
        lambda: 'return id(va).intent_id().find("timer_") == 0;'
      then:
        - script.execute:
            id: mm_timer_voice_handler
            intent_name: !lambda 'return id(va).intent_id();'
            intent_data: !lambda 'return id(va).intent_data();'
```

## Current Workaround

For now, voice intents should be routed through:

1. **Direct Service Calls**: Home Assistant calls `esphome.device_mm_module_voice_handler`
2. **Automations**: Match intents and trigger device services
3. **Custom Integrations**: Build Home Assistant integration to handle routing

This keeps the module system independent of Home Assistant implementation details.
