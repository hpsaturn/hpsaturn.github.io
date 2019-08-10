---
layout: post
title:  "MatrixCreator Android demo via MALOS"
date:   2018-08-02
excerpt: "Android application that interfaces with MATRIX Creator MALOS layer"
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

<!-- 
<div class="row">
  <div class="col-md-4" markdown="1">
  </div>
  <div class="col-md-4" markdown="1">
  <figure>
    <a href="{{ site.url }}/assets/img/matrixcreatorandroid.jpg"><img src="{{ site.url }}/assets/img/matrixcreatorandroid.jpg"></a>
    <figcaption>Android MALOS demo - Main controls.</figcaption>
  </figure>
  </div>
</div>
 -->

## Current Features

* Everloop RGB color control
* Humidity Sensor
* Temperature Sensor
* UV index radation
* IMU (x,y) widget visualitation
* GPIO input/output configuration (pin 0,1)
* GPIO updates callbacks
* Auto discovery Matrix Creator on LAN network
* Manual IP Matrix device target

<iframe width="560" height="315" src="https://www.youtube.com/embed/ihV_v7zFO7A" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

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

