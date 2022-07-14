---
layout: post
title:  "MatrixCreator / MatrixVoice - Admobilize"
imghead: matrixvoice_capture00.jpg
date:   2016-06-03
excerpt: "Android and hardware integrations over MatrixLabs boards"
project: true
tag:
- Android
- PlatformIO
- Alexa
- GoogleHome
- IA
- PocketSphinx
- ESP32
- C++
- NDK
- AndroidThings
- IoT
- Protos
- GRPC
- ZeroMQ
- MatrixVoice
comments: false
---
   
<center><b>MatrixCreator / MatrixVoice</b>Android and hardware integrations over MatrixLabs boards</center>

Together with [Admobilize](https://www.admobilize.com/) team we released two boards in four years that using different technologies. In the main part of time I worked on low level layers of Android with C++ integrations using OpenCV and ZeroMQ techologies. Also we working with some Google technologies like GRPC and Protocol Buffers for try to generate different SDK to some software languages.

---

## Technologies


- [x] Android JNI using CMake variant to NDK
- [x] OpenCV in Android to have a low level camera in Background
- [x] Bluetooth low energy (BLE) GATT sever for communications
- [x] ESP32 integration with PlatformIO CI
- [x] Hal abstraction layer in C++ over Raspbian for each board.
- [x] Multi user - multi language hardware abstraction using ZMQ
- [x] Wakeword implementations on C++ using PocketSphinx
- [x] GRPC and protocol buffers for realtime communication and APIs
- [x] Android Things implementation over RaspberryPi
- [x] Our Balena implementation in parallel to Android Things development

---

<a href="https://youtu.be/YMRRN0Mzvw0" target="_blank"><img src="{{ site.url }}/assets/img/matrixvoice_youtube.jpg" align="center"></a>

[MatrixLabs](https://www.matrix.one/)  
[Admobilize](https://www.admobilize.com/)  
[Github organization](https://github.com/matrix-io/)  
[MatrixVoice posts here](https://hpsaturn.com/tags/#MatrixVoice)


{% capture images %}
  {{ site.url }}/assets/img/matrixcreatorheader.png
  {{ site.url }}/assets/img/matrixvoiceheader.png
  {{ site.url }}/assets/img/matrixcreatorandroid.jpg
  {{ site.url }}/assets/img/admobilize_face_detection.jpg
{% endcapture %}
{% include gallery images=images caption="MatrixCreator / MatrixVoice Gallery" cols=3 %}
