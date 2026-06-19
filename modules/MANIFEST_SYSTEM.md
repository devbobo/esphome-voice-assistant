################################################################################
## Module Manifest System - Implementation Guide
##
## This document explains how Option 4 (Module Manifest System) works in the
## esphome-voice-assistant architecture.
##
## WHAT IS A MODULE MANIFEST?
## ==========================
## A manifest.yaml file that declares:
##   - What voice intents a module handles
##   - What display pages it renders
##   - What LED states it manages
##   - What Home Assistant entities it exposes
##   - What audio sounds it uses
##
## WHY MANIFESTS?
## ==============
## - Declarative: Read the manifest to understand what a module does
## - Self-documenting: No need to hunt through code
## - Discoverable: Can auto-generate module index/documentation
## - Composable: Core managers can build routing tables from manifests
## - Future-proof: Can extend with new integration types
##
## HOW IT WORKS
## ============
## 1. Module Developer:
##    - Creates manifest.yaml declaring capabilities
##    - Implements handler scripts with predictable names
##    - Names follow pattern: mm_{module}_{handler_type}_{handler_id}
##
## 2. At Compile Time:
##    - ESPHome includes module.yaml which includes manifest.yaml
##    - Manifest is available as YAML data (can be referenced in comments/docs)
##    - Core managers include modules and their handlers
##
## 3. At Runtime:
##    - Core managers have generic handlers that look for module scripts
##    - Voice handler calls mm_{intent_module}_voice_handler if it exists
##    - Display handler calls mm_{page_module}_render_{page_name} if it exists
##    - LED/Audio handlers work via shared event buses (no lookup needed)
##
## INTEGRATION PATTERN
## ===================
## Each core manager has a generic extension point that looks for module handlers.
## 
## Example - Voice Manager on_end:
##   - Parse recognized intent from Home Assistant
##   - Extract module name from intent ID (e.g., "timer_*" → module "timer")
##   - Call mm_timer_voice_handler(intent_id, parameters)
##   - If handler doesn't exist, gracefully log and continue
##
## Example - Display Manager when dm_current_view changes:
##   - Check if view starts with "module_" (e.g., "module_timer:countdown")
##   - Extract module name and page ID
##   - Call mm_timer_render_countdown()
##   - If handler doesn't exist, log warning and render empty
##
## Example - LED/Audio:
##   - No lookup needed - modules directly trigger led_event and audio_queue_sound
##   - These are shared infrastructure, not module-specific
##
## NAMING CONVENTIONS
## ==================
## All module artifacts follow the mm_{module_name}_ prefix:
##
## Voice Handlers:
##   - mm_timer_voice_handler(intent_id: string, parameters: string)
##   - Handles all intents declared in manifest's voice_intents
##
## Display Renderers:
##   - mm_timer_render_countdown()
##   - mm_timer_render_expired()
##   - One script per display_page in manifest
##
## State Variables:
##   - mm_timer_duration_sec
##   - mm_timer_is_active
##   - mm_timer_started_ms
##   - mm_timer_is_expired
##
## Commands:
##   - mm_timer_command_set(duration_sec)
##   - mm_timer_command_cancel()
##   - mm_timer_command_acknowledge_expired()
##
## HA Entities:
##   - mm_timer_is_active_ha (binary_sensor)
##   - mm_timer_remaining_seconds (sensor)
##   - mm_timer_duration_setter (number)
##   - mm_timer_button_cancel (button)
##
## MODULE CHECKLIST
## ================
## When implementing a new module:
##
## □ Create manifest.yaml with:
##   □ name, description, version
##   □ voice_intents (with handler_script names)
##   □ display_pages (with render_script names)
##   □ led_states (mapped to led_event types)
##   □ audio_sounds (with IDs and descriptions)
##   □ ha_entities (auto-discovered)
##   □ state_variables (with descriptions)
##
## □ Create core.yaml with:
##   □ All globals matching state_variables
##   □ All command scripts (mm_module_command_*)
##   □ Internal tick/interval logic
##
## □ Create voice.yaml with:
##   □ mm_module_voice_handler script
##   □ Receives intent_id and parameters
##   □ Routes to appropriate command scripts
##
## □ Create display.yaml with:
##   □ mm_module_render_* scripts for each page
##   □ Each handles rendering to LVGL
##
## □ Create ha_entities.yaml with:
##   □ binary_sensor, sensor, number, button
##   □ IDs match manifest's ha_entities section
##
## □ Create module.yaml with:
##   □ Aggregates all above via packages
##
## ENABLING A MODULE
## =================
## To enable a module (e.g., timer) on a device:
##
##   In event_router.yaml packages:
##   ```
##   module_timer: !include modules/timer/module.yaml
##   ```
##
##   That's it! The manifest describes everything, core managers auto-discover.
##
## SCALING
## =======
## This approach scales beautifully:
## - Add new modules: just add new manifest + implementation files
## - No core changes needed per module
## - Multiple modules coexist without conflict (mm_{name}_ prefix prevents clashes)
## - Core managers automatically adapt based on what modules are included
##
## FUTURE EXTENSIONS
## ==================
## Manifest system allows for future additions:
##   - automation_triggers: what events modules can fire
##   - custom_integrations: vendor-specific platforms
##   - dependencies: module A requires module B
##   - conflicts: module A incompatible with module B
##
## This keeps the system open-ended while maintaining clean architecture.
################################################################################
