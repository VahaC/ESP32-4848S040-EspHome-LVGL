# ESP32-4848S040 + ESPHome + LVGL Dashboard

ESPHome configuration for an ESP32-S3 board paired with the 480×480 ST7701S panel (module 4848S040) and a GT911 capacitive touchscreen. The provided YAML renders a polished LVGL dashboard, synchronizes sensor data with Home Assistant, and exposes on-device controls for backlight brightness and text color.

## Features
- 480×480 RGB ST7701S display driven over hardware SPI with octal PSRAM at 80 MHz and fully tuned timing parameters.
- LVGL UI with two pages: a sensor-focused home screen and a configuration screen.
- Native Home Assistant integration through the encrypted `api` for temperature, humidity, pressure, and battery level.
- Exposed monochromatic light entity for the display backlight, real-time brightness sensor, web server, and OTA updates.
- Complete text-color picker (R/G/B sliders + HEX field) that refreshes every widget via `apply_text_color` and `update_color_ui` scripts.
- Auto-return timer for the settings page, synced sliders/labels, and verbose diagnostics via `debug` and `logger`.

## Hardware Requirements
- ESP32-S3 module with PSRAM (e.g., WT32-S3 or similar boards).
- 4.8" 480×480 ST7701S display (4848S040 carrier).
- GT911 capacitive touch controller connected over I²C (GPIO19 SDA, GPIO45 SCL).
- 5 V power source; backlight PWM on GPIO38.
- Home Assistant sensors providing the environmental data shown on screen.

## Pinout and Interfaces
| Function | GPIO |
| --- | --- |
| Backlight (LEDC PWM) | GPIO38 |
| ST7701S SPI CLK / MOSI | GPIO48 / GPIO47 |
| CS | GPIO39 |
| DE / HSYNC / VSYNC | GPIO18 / GPIO16 / GPIO17 |
| PCLK | GPIO21 (12 MHz) |
| Data lines (R,G,B) | R: 11,12,13,14,0; G: 8,20,3,46,9,10; B: 4,5,6,7,15 |
| I²C touch | SDA GPIO19, SCL GPIO45 |

## Required Home Assistant Entities
Pick the Home Assistant entities you want to display (update the `sensor:` block accordingly):

- `sensor.outdoor_sensor_temperature`
- `sensor.outdoor_sensor_humidity`
- `sensor.outdoor_pressure_mmhg`
- `sensor.outdoor_sensor_battery`

## Wi-Fi and Secrets
Create `secrets.yaml` with `wifi_ssid` and `wifi_password`. OTA password and API encryption key are already filled in the YAML (`ota.password`, `api.encryption.key`); change them if you want unique credentials.

## Getting Started
1. Install ESPHome (`pip install esphome`).
2. Clone this repository and open the folder in VS Code.
3. Configure `secrets.yaml` and adjust Home Assistant entity IDs if they differ.
4. Run `esphome run esphome-esp32-s3-4848s040.yaml` to compile and flash (USB or OTA).
5. Once online, the device appears in Home Assistant, exposing the backlight light entity and text-color control via the text sensor.

## UI Overview
- **Home page**: large temperature/humidity widgets, compact pressure and battery cards, and a Home Assistant icon that opens the settings page.
- **Settings page**: brightness slider (5–100 %), R/G/B sliders, HEX label, color preview, and a 30 s idle timer that snaps back to the home page.
- **Text Sensor “Text Color Hex”**: input a HEX string from Home Assistant; the device parses it, updates globals, and re-renders the UI instantly.

## Customization
- Replace Google Fonts in the `font:` section or point to local TTF files.
- Adjust `display:` timing, `rotation`, `data_rate`, or enable `auto_clear_enabled` depending on your panel orientation and requirements.
- Extend `sensor:` or `text_sensor:` with more entities and add widgets/pages inside `lvgl.pages` to visualize additional data.

## License
Distributed under the license specified in the repository’s `LICENSE` file.