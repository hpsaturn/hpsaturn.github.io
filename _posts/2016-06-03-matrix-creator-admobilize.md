---
layout: post
title:  "MatrixCreator / MatrixVoice - Admobilize"
imghead: matrixheader.jpg
date:   2016-06-03
excerpt: "Android and hardware integrations over MatrixLabs boards"
feature: http://hpsaturn.com/assets/img/matrixvoiceheader.png
project: true
tag:
- Android
- PlatformIO
- MatrixVoice
- MatrixCreator
- Alexa
- GoogleHome
- PocketSphinx
- ESP32
- IoT
- C++
- Protobuf
- ZeroMQ
comments: false
---
   
# MatrixCreator / MatrixVoice

**Android and hardware integrations over MatrixLabs boards**

Together with [Admobilize](https://www.admobilize.com/) team we released two boards in four years that using different technologies. In the main part of time I worked on low level layers of Android with C++ integrations using OpenCV and ZeroMQ techologies. Also we working with some Google technologies like GRPC and Protocol Buffers for try to generate different SDK to some software languages.

---

## Technologies

- Android JNI using CMake variant to NDK
- OpenCV in Android to have a low level camera in Background
- Bluetooth low energy (BLE) GATT sever for communications
- ESP32 integration with PlatformIO CI
- Hal abstraction layer in C++ over Raspbian for each board.
- Multi user - multi language hardware abstraction using ZMQ
- Wakeword implementations on C++ using PocketSphinx
- GRPC and protocol buffers for realtime communication and APIs
- Android Things implementation over RaspberryPi
- Our Balena implementation in parallel to Android Things development

![Admobilize Face detection]({{ site.url }}/assets/img/admobilize_face_detection.jpg)

---

<a href="https://youtu.be/YMRRN0Mzvw0" target="_blank"><img src="{{ site.url }}/assets/img/matrixvoice_youtube.jpg" align="center"></a>

## References

[MatrixCreator](https://matrix-io.github.io/matrix-documentation/matrix-creator/overview/)  
[MatrixVoice](https://matrix-io.github.io/matrix-documentation/matrix-voice/overview/)  
[Admobilize](https://www.admobilize.com/)  
[Github organization](https://github.com/matrix-io/)  
[MatrixVoice posts here](https://hpsaturn.com/tags/#MatrixVoice)

{% capture images %}
  {{ site.url }}/assets/img/matrixcreatorheader.png
  {{ site.url }}/assets/img/matrixvoiceheader.png
{% endcapture %}
{% include gallery images=images caption="MatrixCreator / MatrixVoice Gallery" cols=3 %}
