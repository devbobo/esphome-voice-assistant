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

## Alternative: On-Device Routing via Text Matching

ESPHome's `on_stt_end` handler provides access to transcribed text as variable `x`. This enables 
on-device routing without Home Assistant involvement, though it uses pattern matching rather 
than semantic intent matching.

### Implementation

Configure voice assistant with routing logic:

```yaml
voice_assistant:
  microphone: ...
  speaker: ...
  on_stt_end:
    - if:
        condition:
          lambda: 'return x.find("timer") != std::string::npos;'
        then:
          - script.execute:
              id: mm_timer_voice_handler
              voice_text: !lambda 'return x;'
    - if:
        condition:
          lambda: 'return x.find("alarm") != std::string::npos;'
        then:
          - script.execute:
              id: mm_alarm_voice_handler
              voice_text: !lambda 'return x;'
```

Each module parses the voice text to extract parameters:

```yaml
# modules/timer/voice.yaml
script:
  - id: mm_timer_voice_handler
    parameters:
      voice_text: string
    then:
      - if:
          condition:
            lambda: |
              return voice_text.find("set") != std::string::npos ||
                     voice_text.find("start") != std::string::npos;
          then:
            - script.execute:
                id: mm_timer_command_set
                duration_sec: !lambda 'return extract_duration_from_text(voice_text);'
```

### Pros & Cons

**Advantages:**
- Fully on-device (no HA dependency)
- Faster response (no HA round-trip)
- Simpler architecture for simple commands

**Disadvantages:**
- Text pattern matching is fragile across languages and phrasings
- Requires more complex parsing logic in device scripts
- Less flexible than semantic intent matching

## Recommended Approach

For best results, **combine both methods**:

1. Use `on_stt_end` for quick feedback and state changes on-device
2. Use HA automations for complex intent matching and multi-device coordination

This keeps the module system independent while leveraging Home Assistant's strengths.
