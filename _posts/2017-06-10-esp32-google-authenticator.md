---
layout: post
title:  "Google Authenticator Key"
imghead: esp32_potp_horizontal_collage.jpg
date:   2017-06-10
excerpt: "Portable ESP32 POTP key, One-Time Password for two-factor authentication."
project: true
tag:
- ESP32
- Android
- PlatformIO
- Bluetooth
- Arduino
- C++
- Hardware
- IoT
- OpenHardware
- Security
comments: false
---

# Google Authenticator Key

Portable EAP Protected One-Time Password (EAP-POTP). Hardware for provides two-factor user authentication. This is a
PlatformIO project that using `WeMOS` like board with a `ESP32` and OLED SSD1306 display that interfaced with `GoogleAuthenticator` Android app. 

## Development Status: 

<img src="{{ site.url }}/assets/img/esp32_potp_intro.jpg"
srcset="{{ site.url }}/assets/img/esp32_potp_intro.jpg 100w, {{ site.url }}/assets/img/esp32_potp_intro.jpg 200w"
sizes="2vw"
align="right"
alt="image alt text">

- [X] WifiManager
- [X] Wifi OTA Handler
- [X] DeepSleep mode (testing)
- [X] GUI overlay tests
- [X] RFC 4793 implementation
- [ ] Sync RTC via Bluetooth
- [ ] GUI Page viewer for OTP codes
- [ ] Android GoogleAuth POTP sync
- [ ] Vulnerability security test 
- [ ] 3D print case?
- [ ] Resina case?
- [ ] Latex case?

## Technologies

`ESP32`: A Different IoT Power and Performance from [Espressif](https://www.espressif.com/en/products/hardware/esp32/overview) <br/>
`PlatformIO`: is an open source ecosystem for [IoT development](https://platformio.org/) <br/>
`Android`: is the world's most popular mobile platform. <br/>
`RFC 4793 protocol`: Portable EAP Protected One-Time Password (EAP-POTP).

## Compile and firmware upload

Please review the last version of this document in [github](https://github.com/hpsaturn/esp32_wemos_oled.git).

#### Setup WiFi

Setup your WiFi on `secrets.load` and run:

``` bash
git clone https://github.com/hpsaturn/esp32_wemos_oled.git
cd esp32_wemos_oled
cp secrets.load.sample secrets.load
chmod 755 secrets.load
```

#### Compile firmware and deploy

```javascript
platformio run --target upload
``` 

#### OTA Updates (Optional)

For faster firmare updates via air (OTA)

```javascript
platformio run --target upload --upload-port 192.168.1.107
``` 

### Library installation Troubleshooting

if you have the next error:

```bash
src/Main.cpp:27:18: fatal error: sha1.h: No such file or directory
compilation terminated.
```
it is because the TOTP Arduino library is broken, please clean and remove hidden directories with `rm -rf .pioenvs .piolibdeps` and run `git pull origin master` again.

