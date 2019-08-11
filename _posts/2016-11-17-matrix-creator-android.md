---
layout: post
title:  "MatrixCreator Android"
date:   2018-08-02
excerpt: "Android application that interfaces with MATRIX Creator MALOS layer."
feature: http://hpsaturn.com/assets/img/matrixcreatorheader.png
tag:
- MatrixVoice
- MatrixCreator
- Android
- ZeroMQ
- Protobuf
- IoT
- OpenHardware
comments: false
---
  
Android application that interfaces with MATRIX Creator MALOS layer. 

## Technologies

I'm using different technologies in the current project: <br/> 

* `MatrixCreator`: fully-featured [development board](https://www.espressif.com/en/products/hardware/esp32/overview) with different technologies like `Z-Wave`, `zigbee` and `NFC`. Also it include `Xilinx Spartan FPGA`, Microphone Array (8 MEMS audio sensors), Everloop (35 RGBW LEDS) and UV, IR, IMU modules.<br/>
* `MALOS`: MATRIX Creator [abstraction layer](https://github.com/matrix-io/matrix-creator-malos) that using `ZeroMQ` [technology](http://zeromq.org/).
* `Protocol buffers`: are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data.<br/>
* `Android`: is the world's most popular mobile platform. <br/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/ihV_v7zFO7A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Current Features

{% if page.mobile %}
{% else %}
<a href="{{ site.url }}/assets/img/matrixcreatorandroid.jpg"><img src="{{ site.url }}/assets/img/matrixcreatorandroid.jpg" align="right"></a>
{% endif %}

* Everloop RGB color control
* Humidity Sensor
* Temperature Sensor
* UV index radation
* IMU (x,y) widget visualitation
* GPIO input/output configuration (pin 0,1)
* GPIO updates callbacks
* Auto discovery Matrix Creator on LAN network
* Manual IP Matrix device target

## TODO
- [X] GPIO callback
- [X] Pressure
- [ ] Mic Array visualization (beamforming localization)
- [ ] ZigbeeBulb basic control
- [ ] LIRC custom control config
- [ ] RaspberryPi Wifi config via BT4
- [ ] MALOS WakeWord config via Android (in development)

## Preriquisities

* Please install **Matrix Creator CORE** (MALOS service package) on your RaspberryPi and reboot it:

``` bash
curl https://apt.matrix.one/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.matrix.one/raspbian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/matrixlabs.list
sudo apt update
sudo apt upgrade
sudo apt install matrixio-malos
reboot
```

**NOTE**: 
* For more details: [Getting Started Guide](https://matrix-io.github.io/matrix-documentation/matrix-core/getting-started/core-installation/)
* Your creator on the same network
* Android 4.4.x or later

## Download
Pre-releases for testing, please download [here](https://github.com/matrix-io/matrix-creator-malos-android/releases).

## Preriquisities and dependencies for Build

#### Clone repository and submodules

```
git clone --recursive https://github.com/matrix-io/matrix-creator-malos-android.git
cd matrix-creator-malos-android
```

#### Fabric configuration

create file matrix-malos-demo/app/fabric.properties with:
```
apiSecret=<YOUR FABRIC SECRET>
apiKey=<YOUR FABRIC API KEY>
```
(or open your project on AndroidStudio and config crashlytics fabric plugin or remove this dependency on gradle app file)

#### Building and install

```
./gradlew assembleDebug
./gradlew installDebug
```
(or with AndroidStudio IDE)

