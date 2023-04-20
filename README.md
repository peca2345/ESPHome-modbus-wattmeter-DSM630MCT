# ESPHome modbus - 3F DIN wattmeter - DSM630MCT

## Description:
It is a three-phase non-invasive wattmeter with a DIN rail measuring coil. It communicates via modbus protocol via ESP8266/ESP32 to LAN. The program is written using ESPHome and is fully integrated into Home Assistant. It is a cheap alternative to, for example, Shelly 3EM.

![DSM630MCT](https://github.com/peca2345/ESPHome-modbus-wattmeter-DSM630MCT/blob/main/IMG/DSM630MCT_.png?raw=true)

## Info:
- I do not recommend buying an unnecessary large clamp - 100A is really huge!
- use only shielded cable, otherwise the error "Modbus CRC Check Failed!" may appear in the log.
- put a 120 ohm resistor after the last connected device
- modbus datasheet: [DSM630MCT](https://github.com/peca2345/ESPHome-modbus-wattmeter-DSM630MCT/blob/main/IMG/DSM630MCT_3_phase_wattmeter_datasheet.pdf)
- you have to find out what the wattmeter address is - default is 0x01
- you also need to find out the serial port speed - default 9600
- in ESPHome use the sensor class only for addresses that are read-only
- for addresses that are read use the "sensor" class
- for addresses that are write use the "number" class (you can then change their values in lovelace)
- for each register you want to have in HA you have to create a separate sensor in ESPHome
- as long as the address of the device is the same as some sensor, it doesn't matter
- no need to set the wattmeter - works with default address 0x01

## Components:
- ESP8266 / ESP32
- RS485/TTL converter: [SHOP](https://www.laskakit.cz/prevodnik-ttl-na-rs-485--max485/) 
- Wattmeter DSM630MCT: [SHOP](https://www.aliexpress.com/item/1005004059758839.html?dp=60884bf2d2ed5efc7c969b20&cn=ah&aff_fcid=f70ab32472014c52a8e7cedd4dd3921e-1682023557871-06550-_sSETun&aff_fsk=_sSETun&aff_platform=link-c-tool&sk=_sSETun&aff_trace_key=f70ab32472014c52a8e7cedd4dd3921e-1682023557871-06550-_sSETun&terminal_id=153e61cce3cb4f32a9ecb3735d8d5ed6&afSmartRedirect=y) 

## Schematic ESP32:
![Schema](https://github.com/peca2345/ESPHome-modbus-wattmeter-DSM630MCT/blob/main/IMG/schematic2.png?raw=true)

## Schematic ESP8266 - Wemos D1 mini:
![Schema](https://github.com/peca2345/ESPHome-modbus-wattmeter-DSM630MCT/blob/main/IMG/schematic_wemos.png?raw=true)

## Wiring:
![Wiring](https://github.com/peca2345/ESPHome-modbus-wattmeter-DSM630MCT/blob/main/IMG/schema.jpg?raw=true)

## Modbus address:
```
0x01  Wattmeter default address
```

```
0x00  A-phase voltage         XXX.X V 
0x01  B-phase voltage         XXX.X_V 
0x02  C-phase voltage         XXX.X V 
0x03  A phase current         XXX.X A 
0x04  B phase current         XXX.X A 
0x05  C phase current         XXX.X A 
0x06  Neutral current         XXX.X A
0x07  Total active power      XXXX W 
0x08  A-phase active power    XXXX W 
0x09  B-phase active power    XXXX W 
0x0A  C-phase active power    XXXX W 
0x08  Total reactive power    XXXX Var 
0x0C  A-phase reactive power  XXXX Var 
0x0D  B-phase reactive power  XXXX Var 
0x0E  C-Phase reactive power  XXXX Var 
0x0F  Total apparent power    XXXX VA 
0x10  A-phase apparent power  XXXX VA 
0x11  B-phase apparent power  XXXX VA 
0xl2  C-phase apparent power  XXXX VA 
0x13  Total Power Factor      XXXX 
0x14  A-phase power factor    XX.XX 
0x15  B-phase power factar    XX.XX 
0x16  C-phase power factor    XX.XX 
0x17  A phase line voltage    XXX.X V 
0x18  B-phase line voltage    XXX.X V 
0x19  C phase line voltage    XXX.X V 
0x1A  A-phase frequency       XX.XX HZ 
0x1B  B-phase frequency       XX.XX HZ 
0x1C  C-phase frequency       XX.XX HZ 
0x1D  Forward active energy   XX.XX KWH 
0x1E  Forward active energy   XX.XX KWH  
0x1F  Reverse active energy   XX.XX KWH 
0x20  Reverse active energy   XX.XX KWH 
0x21  Forward reactive energy XXX KVarh 
0x22  Forward reactive energy XXX KVarh 
0x23  Reverse reactive eneray XXX KVarh 
0x24  Reverse reactive energy XXX KVarh 
```
## ESPHome code:
```
uart:
  id: mod_bus
  rx_pin: GPIO13
  tx_pin: GPIO16
  baud_rate: 9600
  stop_bits: 1

modbus:
  flow_control_pin: GPIO23 (only if the converter has a flowpin)
  send_wait_time: 200ms
  id: rs485

modbus_controller:
  - id: wattmetr_home
    address: 0x01
    modbus_id: rs485
    # setup_priority: -10
    update_interval: 60s
    
sensor:
  # WATTMETR HOME - A phase voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Voltage"
    address: 0x00
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1 

  # WATTMETR HOME - B phase voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Voltage"
    address: 0x01
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - C phase voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Voltage"
    address: 0x02
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - A phase current
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Current"
    address: 0x03
    unit_of_measurement: "A"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - B phase current
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Current"
    address: 0x04
    unit_of_measurement: "A"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - C phase current
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Current"
    address: 0x05
    unit_of_measurement: "A"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - Neutral current
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Neutral Current"
    address: 0x06
    unit_of_measurement: "A"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # Total active power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Total Active Power"
    address: 0x07
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_WORD

  # A-phase active power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Active Power"
    address: 0x08
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_WORD

  # B-phase active power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Active Power"
    address: 0x09
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_WORD

  # C-phase active power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Active Power"
    address: 0x0A
    unit_of_measurement: "W"
    register_type: holding
    value_type: U_WORD

  # Total reactive power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Total Reactive Power"
    address: 0x08
    unit_of_measurement: "Var"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - A-phase reactive power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Reactive Power"
    address: 0x0C
    unit_of_measurement: "Var"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - B-phase reactive power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Reactive Power"
    address: 0x0D
    unit_of_measurement: "Var"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - C-phase reactive power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Reactive Power"
    address: 0x0E
    unit_of_measurement: "Var"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - Total apparent power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Total Apparent Power"
    address: 0x0F
    unit_of_measurement: "VA"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - A-phase apparent power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Apparent Power"
    address: 0x10
    unit_of_measurement: "VA"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - B-phase apparent power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Apparent Power"
    address: 0x11
    unit_of_measurement: "VA"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - C-phase apparent power
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Apparent Power"
    address: 0x12
    unit_of_measurement: "VA"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - Total Power Factor
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Total Power Factor"
    address: 0x13
    unit_of_measurement: "%"
    value_type: U_WORD
    register_type: holding

  # WATTMETR HOME - A-phase power factor
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Power Factor"
    address: 0x14
    unit_of_measurement: "Hz"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 2

  # WATTMETR HOME - B-phase power factor
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Power Factor"
    address: 0x15
    unit_of_measurement: "Hz"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 2

  # WATTMETR HOME - C-phase power factor
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Power Factor"
    address: 0x16
    unit_of_measurement: "%"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 2

  # WATTMETR HOME - A phase-to-line voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Line Voltage"
    address: 0x17
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - B-phase line voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Line Voltage"
    address: 0x18
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - C-phase line voltage
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Line Voltage"
    address: 0x19
    unit_of_measurement: "V"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.1

  # WATTMETR HOME - A-phase frequency
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-A-phase Frequency"
    address: 0x1A
    unit_of_measurement: "Hz"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.01

  # WATTMETR HOME - B-phase frequency
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-B-phase Frequency"
    address: 0x1B
    unit_of_measurement: "Hz"
    register_type: holding
    value_type: U_WORD
    accuracy_decimals: 1
    filters:
      - multiply: 0.01

  # WATTMETR HOME - C-phase frequency
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-C-phase Frequency"
    address: 0x1C
    unit_of_measurement: "Hz"
    register_type: holding
    value_type: U_WORD    
    accuracy_decimals: 1
    filters:
      - multiply: 0.01

  # WATTMETR HOME - Forward active energy (high 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Forward Active Energy"
    address: 0x1D
    unit_of_measurement: "kWh"
    register_type: holding
    value_type: U_WORD


  # WATTMETR HOME - Forward active energy (lower 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Forward Active Energy"
    address: 0x1E
    unit_of_measurement: "kWh"
    register_type: holding
    value_type: U_WORD


  # WATTMETR HOME - Reverse active energy (high 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Reverse Active Energy"
    address: 0x1F
    unit_of_measurement: "kWh"
    register_type: holding
    value_type: U_WORD


  # WATTMETR HOME - Reverse active energy (lower 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Reverse Active Energy"
    address: 0x20
    unit_of_measurement: "kWh"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - Forward reactive energy (high 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Forward Reactive Energy"
    address: 0x21
    unit_of_measurement: "kVarh"
    register_type: holding
    value_type: U_WORD


  # WATTMETR HOME - Forward reactive energy (lower 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Forward Reactive Energy"
    address: 0x22
    unit_of_measurement: "kVarh"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - Reverse reactive energy (high 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Reverse Reactive Energy"
    address: 0x23
    unit_of_measurement: "kVarh"
    register_type: holding
    value_type: U_WORD

  # WATTMETR HOME - Reverse reactive energy (lower 16 bits)
  - platform: modbus_controller
    modbus_controller_id: wattmetr_home
    name: "wattmeter-home-Reverse Reactive Energy"
    address: 0x24
    unit_of_measurement: "kVarh"
    register_type: holding
    value_type: U_WORD
```

## Lovelace code:
```
type: entities
entities:
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_active_power
    name: A phase power
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_apparent_power
    name: A phase apparent power
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_current
    name: A phase current
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_frequency
    name: A phase frequency
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_line_voltage
    name: A phase line voltage
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_power_factor
    name: A phase power factor
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_reactive_power
    name: A phase reactive power
  - entity: sensor.kc868_e16s_wattmeter_home_a_phase_voltage
    name: A phase voltage
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_active_power
    name: B phase power
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_apparent_power
    name: B phase apparent power
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_current
    name: B phase current
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_frequency
    name: B phase frequency
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_line_voltage
    name: B phase line voltage
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_power_factor
    name: B phase power factor
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_reactive_power
    name: B phase reactive power
  - entity: sensor.kc868_e16s_wattmeter_home_b_phase_voltage
    name: B phase voltage
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_active_power
    name: C phase power
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_apparent_power
    name: C phase apparent power
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_current
    name: C phase current
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_frequency
    name: C phase frequency
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_line_voltage
    name: C phase line voltage
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_power_factor
    name: C phase power factor
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_reactive_power
    name: C phase reactive power
  - entity: sensor.kc868_e16s_wattmeter_home_c_phase_voltage
    name: C phase voltage
  - entity: sensor.kc868_e16s_wattmeter_home_forward_active_energy
    name: Forward active energy
  - entity: sensor.kc868_e16s_wattmeter_home_forward_reactive_energy
    name: Forward reactive energy
  - entity: sensor.kc868_e16s_wattmeter_home_neutral_current
    name: Neutral current
  - entity: sensor.kc868_e16s_wattmeter_home_reverse_active_energy
    name: Reverse active energy
  - entity: sensor.kc868_e16s_wattmeter_home_reverse_reactive_energy
    name: Reverse reactive energy
  - entity: sensor.kc868_e16s_wattmeter_home_total_active_power
    name: Total active power
  - entity: sensor.kc868_e16s_wattmeter_home_total_apparent_power
    name: Total apparent power
  - entity: sensor.kc868_e16s_wattmeter_home_total_power_factor
    name: Total power factor
  - entity: sensor.kc868_e16s_wattmeter_home_total_reactive_power
    name: Total reactive power
```
