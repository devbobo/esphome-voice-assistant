# EchoEar Device Configuration

ESP32-S3 based voice assistant device with 360×360 round touchscreen, gesture detection, battery monitoring, and audio I/O.

## Attribution

This configuration is based on the work by **RealDeco** at [xiaozhi-esphome](https://github.com/RealDeco/xiaozhi-esphome/tree/main/devices/Espressif). The original monolithic configuration has been refactored into a modular architecture following separation of concerns principles.

## Hardware Specifications

- **MCU**: ESP32-S3 (240MHz dual-core, 32MB flash, 8MB PSRAM octal mode @ 80MHz)
- **Display**: JC3636W518V2 (360×360 round MIPI SPI touchscreen, manual update)
- **Touchscreen Controller**: CST816 (I2C, GPIO10 interrupt)
- **Audio ADC**: ES7210 (I2S input, 16-bit 16kHz mono, GPIO3)
- **Audio DAC**: ES8311 (I2S output, 48kHz mono, GPIO41)
- **Battery Monitor**: BQ27220 (I2C 0x55, register 0x08 for voltage in mV)
- **Status LED**: GPIO43 (green, inverted logic)
- **Screen Backlight**: GPIO44 (LEDC PWM control)
- **Touch Pads**: GPIO6 (back), GPIO7 (head) - ESP32 capacitive sensing

### I/O Buses

- **I2C (bus_a)**: SCL=GPIO1, SDA=GPIO2, 100kHz (CST816, BQ27220, codecs)
- **I2S**: LRCLK=GPIO39, BCLK=GPIO40, MCLK=GPIO42 (ES7210, ES8311)
- **SPI (display_qspi)**: Quad mode, CLK=18, data=[46,13,11,12]
- **Power**: GPIO9 (vcc3v3_sw), GPIO48 (codec_3v3)

## Device Configuration Files

- **`device.yaml`** - Orchestrator, bundles all EchoEar modules
- **`hardware.yaml`** - Raw ESP32-S3 setup, I/O buses, audio codecs
- **`io_entities.yaml`** - GPIO/audio abstractions (LEDs, speakers, sensors)
- **`battery.yaml`** - BQ27220 I2C polling and calibration
- **`touchscreen.yaml`** - CST816 driver with gesture detection
- **`i18n.yaml`** - Device-specific entity labels and descriptions

## Gesture Detection

The EchoEar uses a CST816 touchscreen controller with sophisticated gesture detection:

### Detection Phases

1. **Touch Start** (`on_touch` handler - `includes/touchscreen_on_touch.yaml`)
   - Records start position and timestamp
   - Initializes movement tracking

2. **Movement Monitoring** (`on_update` handler - `includes/touchscreen_on_update.yaml`)
   - Tracks displacement from start position
   - Updates position for velocity calculation

3. **Classification** (`on_release` handler - `includes/touchscreen_on_release.yaml`)
   - Calculates total displacement and duration
   - Normalizes coordinates for display rotation (0°, 90°, 180°, 270°)
   - Classifies as one of 7 gesture types

### Gesture Classification

- **Swipe**: Movement > 45 pixels (screen_min_dim/8) within 1500ms
- **Long Press**: Hold ≥500ms with movement ≤ max(14, min_dim/12) pixels
- **Double Tap**: Two taps within 400ms window, minimal movement
- **Single Tap**: Quick release with minimal movement or forgiving threshold within 700ms

### Rotation Normalization

Gesture coordinates are automatically adjusted for display rotation:
- Thresholds adapt to screen dimensions (360×360)
- Swipe directions normalized based on rotation setting in `base.yaml`

## Battery Monitoring

The EchoEar includes a BQ27220 fuel gauge for accurate battery voltage and percentage tracking.

### Hardware

- **Sensor**: BQ27220 on I2C bus (address 0x55)
- **Voltage Register**: 0x08 (little-endian, value in mV)
- **Update Interval**: 60 seconds
- **Connected to**: I2C bus_a (SCL=GPIO1, SDA=GPIO2)

### Monitoring Features

- **Voltage Sensor**: Raw voltage reading from BQ27220 register 0x08
- **Percentage Sensor**: Linear calibration mapping 2.8V→0%, 4.2V→100%
- **Redundancy Check**: `last_battery_percent` global prevents duplicate updates
- **Status Tracking**: Charging/Discharging/Full/Empty states

### Calibration

The battery percentage is calculated via linear mapping:
```
Voltage 2.8V → 0% (minimum)
Voltage 4.2V → 100% (maximum)
```

Adjust calibration in `battery.yaml` if your battery has different voltage ranges.

## Customization

### Display Rotation

Display rotation is configured at the **project level** in `base.yaml`:

```yaml
substitutions:
  rotate_display: "0"  # or 90, 180, 270
```

This adjusts gesture detection coordinates for different screen mounting angles.

### Gesture Timings

Gesture timing thresholds are configured at the **project level** in `base.yaml` and apply to all devices:

```yaml
substitutions:
  long_press_duration: "500"      # Hold time for long press (ms)
  double_press_window: "400"      # Window for double-tap detection (ms)
```

To adjust globally, edit these values in `base.yaml`.

### Battery Calibration

If your battery has a different voltage range, edit datapoints in `battery.yaml`:
```yaml
calibrate_linear:
  datapoints:
    - 2.80 -> 0.0      # 2.8V = 0% (dead)
    - 4.20 -> 100.0    # 4.2V = 100% (full)
    # Add more datapoints as needed
```

### Touch Sensitivity

Adjust capacitive touch thresholds in `io_entities.yaml`:
```yaml
esp32_touch:
  setup_mode: false
  measurement_duration: 0.25ms
  sleep_duration: 0.5ms
```

And per-pad thresholds:
```yaml
binary_sensor:
  - platform: esp32_touch
    threshold: 2000  # Increase for less sensitivity, decrease for more
```

## Supported Features

- ✅ **Gesture Detection**: Swipe (4 directions), Long Press, Double Tap, Single Tap
- ✅ **Battery Monitoring**: Voltage and percentage with 11-point calibration
- ✅ **Master Gesture Control**: Switch to enable/disable touchscreen events
- ✅ **Capacitive Touch Pads**: Back and head touch inputs
- ✅ **Audio I/O**: Microphone and speaker abstractions
- ✅ **Status LED**: Green LED control
- ✅ **Screen Control**: Backlight PWM brightness adjustment

## Troubleshooting

**Gesture detection not working**:
- Check GPIO10 CST816 interrupt connection
- Verify I2C bus_a is working (CST816 address 0x68 should appear on scan)
- Adjust touch sensitivity threshold if needed

**Battery sensor not updating**:
- Check I2C bus_a connectivity
- Verify BQ27220 address 0x55 appears on I2C scan
- Try increasing poll interval if readings are erratic

**Display not rotating**:
- Update both `rotate_display` substitution AND display platform rotation
- Gesture thresholds will auto-scale, but verify swipes respond correctly

**Touch pad false triggers**:
- Increase `threshold` value in binary_sensor config
- Adjust measurement/sleep duration for different environmental conditions
