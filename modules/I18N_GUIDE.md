################################################################################
## Module Internationalization (i18n) Guide
##
## Best practices for managing user-facing strings in modules.
## All strings shown to users should be centralized for easy translation.
################################################################################

## OVERVIEW
## ========
## Each module should have an i18n.yaml file containing all user-facing strings.
## This enables:
##   - Easy translation to multiple languages
##   - Consistent terminology across the module
##   - Centralized maintenance
##   - String reuse across different contexts
##
## CURRENT STATE
## =============
## Timer module has i18n.yaml with:
##   - Logger messages (non-critical, for debugging)
##   - Display text (shown on screen to user)
##   - Voice command descriptions
##   - HA entity names
##   - Module metadata
##
## BEST PRACTICES
## ==============

### 1. Static Strings Only
  ✅ GOOD: Substitutions for static text
     name: "${mm_timer_ha_active_binary_sensor}"
     
  ❌ DON'T: Use substitutions for format strings with placeholders
     format: "${mm_timer_log_display_remaining}"  # Contains %u:%02u - won't work!
  
  Use format strings directly in code:
     ESP_LOGI("timer", "Display: %u:%02u remaining", min, sec);

### 2. Naming Convention
  Pattern: ${mm_{module}_{category}_{context}}
  
  Examples:
    ${mm_timer_log_setting_timer_minutes}       # Log message
    ${mm_timer_display_title_countdown}         # Display title
    ${mm_timer_voice_set_minutes_desc}          # Voice intent desc
    ${mm_timer_ha_active_binary_sensor}         # HA entity name

### 3. String Categories
  
  **logs:** Technical/developer-facing
    - Logger messages for debugging
    - Should NOT be translated (often technical)
    - Use sparingly - most logs are for developers
  
  **display:** User-facing screen text
    - Titles, labels, prompts on device
    - Should be translated for each language
    - Keep concise (limited screen space)
  
  **voice:** Voice command descriptions
    - Descriptions of what intents do
    - Shown in Home Assistant docs
    - Should be user-friendly
  
  **ha:** Home Assistant integration
    - Entity names
    - Entity descriptions
    - Button/sensor labels
  
  **module:** Metadata
    - Module name and description
    - Shown in module documentation

### 4. Translation Process
  
  For Spanish:
    1. Copy i18n.yaml → i18n_es.yaml
    2. Translate all substitution values
    3. Include in module.yaml: timer_i18n_es: !include i18n_es.yaml
    4. Device config selects language:
       substitutions:
         language: "es"
       packages:
         timer_i18n: !include modules/timer/i18n_${language}.yaml
  
  For German:
    1. Copy i18n.yaml → i18n_de.yaml
    2. Translate values
    3. Include appropriately

### 5. String Reuse
  
  Define once, use many times:
    ${mm_timer_display_title_countdown}  # Used in:
      - Display render script
      - Voice response
      - HA documentation
  
  This ensures consistent terminology and makes translation easier.

### 6. Context-Specific Variants
  
  If you need variations, use context in the name:
    ${mm_timer_display_prompt_countdown}  # "Say 'cancel' to stop"
    ${mm_timer_display_prompt_expired}    # "Say 'stop' to acknowledge"
  
  NOT: ${mm_timer_display_prompt}  (ambiguous, which prompt?)

### 7. Editor Support
  
  To enable autocomplete in VS Code:
    1. Add .vscode/settings.json:
       ```json
       {
         "yaml.schemas": {
           "file:///path/to/module-i18n-schema.json": "**/modules/**/i18n*.yaml"
         }
       }
       ```
    2. Create module-i18n-schema.json with all valid substitution keys
    3. Editor will highlight typos

### 8. Testing Translations
  
  To test Spanish variant:
    1. In device config, add: language_override: "es"
    2. Compile and test
    3. Verify all strings render correctly
    4. Check for text overflow on display

## FILES STRUCTURE
## ================
modules/timer/
  ├── i18n.yaml              # English (default)
  ├── i18n_es.yaml           # Spanish
  ├── i18n_de.yaml           # German  ├── i18n_fr.yaml           # French (if needed)
  └── I18N_GUIDE.md          # This file

## MODULE.YAML INTEGRATION
## =========================
Include i18n file in module.yaml:

packages:
  # Internationalization - select based on language
  timer_i18n: !include i18n.yaml
  
  # Or with dynamic selection:
  # timer_i18n: !include i18n_${device_language}.yaml

Then use substitutions throughout module:

manifest.yaml:
  name: "${mm_timer_module_name}"
  description: "${mm_timer_module_description}"

ha_entities.yaml:
  - name: "${mm_timer_ha_active_binary_sensor}"
  - name: "${mm_timer_ha_cancel_button}"

## FUTURE ENHANCEMENTS
## ====================
- [ ] Implement translation management tool (pull strings, export for translation)
- [ ] Add automatic pluralization support (e.g., "1 minute" vs "5 minutes")
- [ ] Support for right-to-left languages (Arabic, Hebrew)
- [ ] Voice synthesis language selection based on i18n
- [ ] Unit formatting (time zones, date formats) based on locale
