# EspHome-TDS-and-Flow-meter

ESPHome configuration for an ESP32-C6 based reverse osmosis monitor that reads Total Dissolved Solids (TDS) before and after the RO membrane, water temperatures via NTCs, and live/total flow using a YF-S402B hall-effect flow meter. The main configuration lives in `esp32c6-tds-and-flow-meter.yaml`.

## Features
- Two analog TDS probes (in/out) with temperature compensation and smoothing
- Dual 10 kΩ NTC probes for per-line temperature monitoring
- Flow rate (L/min) and accumulated consumption (L) with flash-backed storage
- Debug text sensors for reset reason and device diagnostics
- ESP-IDF based firmware targeting `esp32-c6-devkitc-1`

## Hardware Summary
| Component | Notes |
| --- | --- |
| MCU | ESP32-C6-DevKitC-1 (ESPHome, ESP-IDF) |
| TDS probes | DFRobot-style analog probes with on-board signal conditioning |
| Temperature | 10 kΩ NTCs wired in a voltage divider (downstream config) |
| Flow sensor | YF-S402B (yellow data wire on GPIO6, internal pull-up) |
| Power | 3.3 V reference for all analog sections |

### Pin Assignments
- GPIO3 → `tds_in_voltage`
- GPIO2 → `tds_out_voltage`
- GPIO4 → `ntc_in_adc`
- GPIO5 → `ntc_out_adc`
- GPIO6 → Flow sensor pulse input

## Home Assistant Entities
| Entity | Description |
| --- | --- |
| `sensor.tds_in` | Compensated TDS before RO membrane (ppm) |
| `sensor.tds_out` | Compensated TDS after RO membrane (ppm) |
| `sensor.tds_in_temperature` | °C from NTC probe inside feed line |
| `sensor.tds_out_temperature` | °C from NTC probe downstream |
| `sensor.water_in_flow_rate` | Instantaneous flow in L/min |
| `sensor.water_in_total` | Integrated total water usage in liters (persistent) |
| `text_sensor.device_info` | Chip model, flash size, etc. |
| `text_sensor.reset_reason` | Last reboot reason |

## Getting Started
1. Install ESPHome (`pip install esphome`) if you have not already.
2. Copy `esp32c6-tds-and-flow-meter.yaml` into your ESPHome project folder.
3. Create or update `secrets.yaml` with:
	```yaml
	wifi_ssid: "YOUR_WIFI"
	wifi_password: "YOUR_PASSWORD"
	```
	Replace the placeholder values with your network credentials.
4. (Optional) Update the `api` encryption key and OTA password in the YAML file.
5. Compile and upload:
	```powershell
	esphome run esp32c6-tds-and-flow-meter.yaml
	```
	ESPHome will build the ESP-IDF firmware and prompt you to connect the ESP32-C6 over USB for the initial flash.

## Calibration Notes
- **TDS**: The template sensors use the DFRobot polynomial with a calibration multiplier (`CAL_FACTOR_IN/OUT = 0.727`). Adjust these constants if your reference solution reports a different ppm.
- **Temperature**: Update the NTC calibration block (`b_constant`, `reference_temperature`, `reference_resistance`) to match your thermistors.
- **Flow**: The default conversion (`multiply: 0.00061086`) assumes ~1637 pulses per liter. Measure 1 L of water, compare against `sensor.water_in_total`, and tweak this value until it matches.

## Troubleshooting
- Ensure the flow sensor data line has a solid pull-up to 3.3 V; the config enables the internal pull-up, but long cables may require an external resistor.
- If ADC readings look noisy, verify that your sensor boards share ground and keep analog wires short.
- Use the ESPHome dashboard logs (`esphome logs esp32c6-tds-and-flow-meter.yaml`) to watch raw voltage sensors when tuning filters.