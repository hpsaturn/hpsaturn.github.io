---
layout: post
title:  "Self Balancing Robot"
date:   2016-04-03
excerpt: "Teensy32 implementation of a Self Balancing Robot."
feature: https://raw.githubusercontent.com/HackBo/Self-Balancing-Robot/master/images/photo_robot.jpg
tag:
- ESP32
- PlatformIO
- Arduino
- Teensy32
- C++
- IoT
- OpenHardware
comments: false
---

# Self Balancing Robot

**Source code**: [Github](https://github.com/HackBo/Self-Balancing-Robot)


##### Initial technologies and hardware:

- Base CPU: [Teensy32](https://www.pjrc.com/teensy/teensy31.html)
- Comm and BUS network: [ESP8266](https://espressif.com/en/products/hardware/esp8266ex/overview)
- Balancing Sensors: [IMU6050](http://www.aliexpress.com/item/MPU-6050-3-Axis-gyroscope-acce-lerometer-module-3V-5V-compatible-For-Ar/1858984311.html)
- Motor Driver: [L298N](http://www.aliexpress.com/item/Free-Shipping-1PCS-New-Dual-H-Bridge-DC-Stepper-Motor-Drive-Controller-Board-Module-L298N-for/32556583041.html)
- Motors: [Basic DC Motors](http://www.aliexpress.com/store/product/HK-POST-FREE-Wholesale-48-1-Plastic-DC-Drive-Gear-Motor-Tyre-Tire-Wheel-For/2035033_32603795906.html)
- Tools and software [PlatformIO framework](http://platformio.org/)

##### Alpha Test Boards:

- NodeMCU 1.0 (ESP-12E Module) (ESP8266)
- Teensy32
- RaspberryPi 3

## Objectives:

- Two PID stages (accelerometer and pitch control)
- ESP8266 basic control (MiniOSC)
- ESP8266 to RaspberryPi Wifi interface via CoAP or OSC protocols (Bus data, control, others)
- RaspberryPi+Camera+ServoMotors (image processing via OpenCV)
- Suggest others features!

<iframe width="560" height="315" src="https://www.youtube.com/embed/7tfVts636bs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
 
## Current main config:

Teensy32 (Arduino framework)

```python
[env:teensy31]
platform = teensy
framework = arduino
board = teensy31
targets = upload
#build_flags = -DTEENSY31 -UUSB_SERIAL -DUSB_SERIAL_HID
```

## Firmware Installation

### Current firmware (Teensy32)

First install PlatformIO via `pip`
 
``` bash
$ pip install -U platformio 
```
or via IDE. More info in the oficial documentation: [http://docs.platformio.org/en/latest/installation.html](http://docs.platformio.org/en/latest/installation.html)

Compile project and deploy

``` bash 
$ platformio run
```

### Firmware v1.0 for Arduino ProMini

From main source and previous steps

```
$ git checkout tags/v1.0-ArduinoProMini
$ platformio run --target clean
$ platformio run
```
   
## Schematics

#### Initial basic robot (ArduinoProMini)

(Alpha version, click for last schematic update)
 
[![Click for last schematic update](https://raw.githubusercontent.com/HackBo/Self-Balancing-Robot/master/images/schematics_basic_self_balancing.png)](http://www.schematics.com/project/self-balancing-robot-31896/)

#### Teensy32 and ESP8266

(cooming soon)


