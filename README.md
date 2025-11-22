# ESP32-4848S040 + ESPHome + LVGL Dashboard

An ESPHome configuration for the ESP32-S3 + 4.8" 480×480 ST7701S (4848S040) module with a GT911 capacitive touchscreen. The YAML file builds a two-page LVGL dashboard that pulls sensor data from Home Assistant, exposes backlight and color controls, and serves a web interface for quick diagnostics.

## Overview
- Targets `esp32s3` + ESP-IDF with octal PSRAM at 80 MHz for the ST7701S RGB panel.
- Boots into a high-contrast dashboard showing temperature, humidity, pressure, and battery level.
- A settings page lets you fine-tune the backlight brightness (PWM via LEDC) and text color (RGB sliders + HEX input).
- Integrates natively with Home Assistant by streaming sensor states through the encrypted ESPHome API.
- Includes OTA upgrades, a fallback AP, a web server, and verbose debug logging for field diagnostics.

## Highlights
- **Display pipeline**: ST7701S RGB (480×480), MODE3 SPI at 2 MHz for commands, PCLK 12 MHz for pixel data, custom porch/pulse timings, and LVGL double buffering (25% of RAM).
- **Touch**: GT911 I²C touchscreen tied to the same LVGL instance for smooth page navigation.
- **Backlight control**: LEDC PWM on GPIO38 exposed as a standard ESPHome light; brightness is synchronized with an LVGL slider and a template sensor (`Current Brightness`).
- **Dynamic theming**: Three global ints keep the active text color; scripts `apply_text_color` and `update_color_ui` repaint every label, bar, and slider knob whenever RGB sliders or the HEX text sensor change.
- **Home Assistant data**: Sensors for temperature, humidity, pressure, and battery read HA entities and immediately update LVGL widgets via `lvgl.label.update` / `lvgl.bar.update` actions.
- **UX niceties**: 30-second idle timer auto-returns from settings to the main page; Home Assistant icon acts as the navigation button in both directions.

## Hardware Checklist
- ESP32-S3 dev board with PSRAM (WT32-S3 or similar) and reliable 5 V power.
- 4.8" 480×480 ST7701S panel (a.k.a. 4848S040) with RGB bus breakout.
- GT911 capacitive touch FPC connected to the I²C pins listed below.
- Optional case/frame to keep the flat-flex cables secure.

## Pin Mapping
| Function | GPIO |
| --- | --- |
| Backlight PWM (LEDC) | 38 |
| ST7701S SPI CLK / MOSI | 48 / 47 |
| ST7701S CS | 39 |
| ST7701S DE / HSYNC / VSYNC | 18 / 16 / 17 |
| ST7701S PCLK | 21 (12 MHz) |
| RGB data lines | R: 11,12,13,14,0; G: 8,20,3,46,9,10; B: 4,5,6,7,15 |
| GT911 SDA / SCL | 19 / 45 |

Feel free to adjust the GPIO map in `esphome-esp32-s3-4848s040.yaml` if your carrier board differs.

## Home Assistant Integration
Pick whichever Home Assistant sensors you want to display and mirror their entity IDs under the `sensor:` block in the YAML. The default entities are:
- `sensor.outdoor_sensor_temperature`
- `sensor.outdoor_sensor_humidity`
- `sensor.outdoor_pressure_mmhg`
- `sensor.outdoor_sensor_battery`

There is also a text sensor (`text_color_hex`) that accepts HEX color strings from Home Assistant, and a template sensor (`Current Brightness`) that publishes actual backlight brightness so Lovelace sliders stay in sync.

## Networking and Secrets
- Add `wifi_ssid` and `wifi_password` to `secrets.yaml` (same folder as the YAML file).
- Customize `api.encryption.key` and `ota.password` if you want unique credentials.
- Fallback AP SSID/password are `Esphome-Esp32-S3-4848S040` / `AWwK5mCFhfZC` if Wi-Fi fails.

## Build & Flash
1. Install ESPHome: `pip install --upgrade esphome` (Python 3.11 recommended).
2. Clone this repository and open it in VS Code.
3. Edit `secrets.yaml` and the Home Assistant entity IDs as needed.
4. Connect the ESP32-S3 board over USB (or ensure OTA is reachable).
5. Run `esphome run esphome-esp32-s3-4848s040.yaml` to compile and flash.
6. After the first boot, the device registers itself with Home Assistant via the ESPHome integration.

## LVGL UI Walkthrough
- **Main page**
	- Large temperature and humidity groups with icons (Material Design icons rendered via LVGL images).
	- Pressure and battery cards along the bottom corners; the battery bar mirrors the Home Assistant percentage.
	- Home Assistant logo at the bottom jumps to the settings page.
- **Settings page**
	- Backlight slider (5–100 %) backed by the same light entity, ensuring the hardware slider and Home Assistant stay aligned.
	- RGB sliders, decimal labels, HEX preview, and a color swatch, all updated through the `update_color_ui` script.
	- Touching any control restarts the idle timer; after 30 s of inactivity the UI returns to the main page with a slide animation.

## Customization Ideas
- Replace Google Fonts (`gfonts://Montserrat`) with any supported Google Font or local TTF file.
- Enable `display.rotation` if your hardware mounts the panel differently (e.g., 270° for USB-down orientation).
- Increase `display.data_rate` / `pclk_frequency` only if your wiring is short and stable.
- Add more LVGL widgets by extending `lvgl.pages` and binding them to new Home Assistant sensors or ESPHome components.
- Hook additional scripts into the existing globals for theming or notifications.

## Troubleshooting
- Use the built-in `web_server` (port 80) to check logs and sensor states if the device is not visible in Home Assistant.
- The `debug` component publishes reset reasons and useful runtime info as text sensors.
- If flickering occurs, lower `ledc` PWM frequency or re-check the RGB pin wiring for the ST7701S bus.

## License
See `LICENSE` for distribution terms.