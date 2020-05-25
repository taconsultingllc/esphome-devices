---
title: Kogan SmarterHome Smart Plug With Energy Meter and 5V 2.4A USB Ports
Model: KASPEMHUSBA
date-published: 2020-05-25
type: plug
standard: au
---
  ![alt text](kogan-smarterhome-smart-plug-energy-meter-5v-24a-usb-ports.jpg "Product Image")

[https://www.kogan.com/au/buy/kogan-smarterhome-smart-plug-energy-meter-5v-24a-usb-ports/](https://www.kogan.com/au/buy/kogan-smarterhome-smart-plug-energy-meter-5v-24a-usb-ports/)

## GPIO Pinout

| Pin    | Function                   |
|--------|----------------------------|
| GPIO03 | Push Button                |
| GPIO13 | Green LED (Inverted: true) |
| GPIO14 | Relay                      |
| GPIO12 | HLW8012 SEL Pin            |
| GPIO04 | HLW8012 CF Pin             |
| GPIO05 | HLW8012 CF1 Pin            |

## Basic Config

```yaml
substitutions:
  device_name: kogan_plug_1
  device_ip: 192.168.x.x
  device_icon: mdi:power-socket-au
  device_restore: ALWAYS_ON
  
  # Higher value gives lower watt readout
  current_res: "0.00225"
  # Lower value gives lower voltage readout
  voltage_div: "805"

esphome:
  name: ${device_name}
  platform: ESP8266
  board: esp8285

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  manual_ip:
    static_ip: ${device_ip}
    gateway: 192.168.x.x
    subnet: 255.255.255.0

logger:
  
api:
  reboot_timeout: 15min
  password: !secret api_password

ota:
  password: !secret ota_password

binary_sensor:
  - platform: gpio
    pin:
      number: 03
      mode: INPUT_PULLUP
      inverted: true
    name: "${device_name}_button"
    on_press:
      - switch.toggle: relay

  - platform: status
    name: "${device_name}_status"

switch:
  - platform: gpio
    id: led
    pin:
      number: GPIO13
      inverted: true

  - platform: gpio
    name: "${device_name}_plug"
    pin: GPIO14
    id: relay
    icon: ${device_icon}
    restore_mode: ${device_restore}
    on_turn_on:
      - switch.turn_on: led
    on_turn_off:
      - switch.turn_off: led

sensor:
  - platform: hlw8012
    sel_pin:
      number: GPIO12
      inverted: true
    cf_pin: GPIO04
    cf1_pin: GPIO05
    current:
      name: "${device_name}_current"
      unit_of_measurement: A
    voltage:
      name: "${device_name}_voltage"
      unit_of_measurement: V
    power:
      id: ${device_name}_wattage
      name: "${device_name}_wattage"
      unit_of_measurement: W
    current_resistor: ${current_res}
    voltage_divider: ${voltage_div}
    change_mode_every: 8
    update_interval: 15s

  - platform: total_daily_energy
    name: "${device_name}_daily_energy"
    power_id: ${device_name}_wattage
    filters:
      - multiply: 0.001
    unit_of_measurement: kWh

  - platform: wifi_signal
    name: "${device_name}_rssi"
    update_interval: 5min

  - platform: uptime
    id: uptime_sec
    name: "${device_name}_uptime"
    update_interval: 5min

text_sensor:
  - platform: template
    name: "${device_name}_upformat"
    lambda: |-
      uint32_t dur = id(uptime_sec).state;
      int dys = 0;
      int hrs = 0;
      int mnts = 0;
      if (dur > 86399) {
        dys = trunc(dur / 86400);
        dur = dur - (dys * 86400);
      }
      if (dur > 3599) {
        hrs = trunc(dur / 3600);
        dur = dur - (hrs * 3600);
      }
      if (dur > 59) {
        mnts = trunc(dur / 60);
        dur = dur - (mnts * 60);
      }
      char buffer[17];
      sprintf(buffer, "%ud %02uh %02um %02us", dys, hrs, mnts, dur);
      return {buffer};
    icon: mdi:clock-start
    update_interval: 5min

time:
  - platform: homeassistant
    id: homeassistant_time
```