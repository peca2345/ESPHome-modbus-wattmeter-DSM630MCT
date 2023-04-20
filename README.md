# ESPHome modbus - 3F DIN wattmeter - DSM630MCT

## Description:
It is a three-phase non-invasive wattmeter with a DIN rail measuring coil. It communicates via modbus protocol via ESP8266/ESP32 to LAN. The program is written using ESPHome and is fully integrated into Home Assistant. It is a cheap alternative to, for example, Shelly 3EM.

## Info:
- use only shielded cable, otherwise the error "Modbus CRC Check Failed!" may appear in the log.
- put a 120 ohm resistor after the last connected device
- modbus datasheet: [Gree Versati III](https://github.com/peca2345/ESPHome-modbus-heatpump-Gree-Versati-III/blob/main/modbus-versati-iii-en.pdf)
- you have to find out what the wattmeter address is - default is 0x01
- you also need to find out the serial port speed - default 9600
- in ESPHome use the sensor class only for addresses that are read-only
- for addresses that are read use the "sensor" class
- for addresses that are write use the "number" class (you can then change their values in lovelace)
- for each register you want to have in HA you have to create a separate sensor in ESPHome
-as long as the address of the device is the same as some sensor, it doesn't matter
