---
layout: post
title:  "CryptoPanel"
date:   2021-09-19
excerpt: "ESP32-Powered eInk Panel for Cryptocurrency Updates and News"
feature: http://hpsaturn.com/assets/img/crypto_panel_preview.jpg
tag:
- PlatformIO
- Arduino
- C++
- OpenHardware
- API
- Python
comments: false
---

# Crypto Panel

ESP32-Powered eInk Panel for Cryptocurrency Updates and News, using a LilyGo EDP47 board.

## Features

- Panel installation and configuration via an easy [Web installer](https://hpsaturn.com/crypto-panel-installer/)
- Configuration via command line (CLI) using builtin console on the web installer
- Support hundreds of coins from [Coingecko API](https://api.coingecko.com/api/v3/coins/list?include_platform=false)
- Random cryptocurrencies news from Cointelegrah and others news portals
- The coin news are follow via QR code
- Base currency USD/EUR configurable
- Firmware update via automatic OTA updates
- Deep sleep configurable (default: 10min).
- Panel temperature ambient configurable (for improve colors)
- Included optional basic 3D-Print frame

## Firmware install

You able to install this firmware on only one click, without compiling nothing only using this [Web installer](https://hpsaturn.com/crypto-panel-installer/). Also you can follow the full instruccions and steps on the [Hackaday Instructions Section](https://hackaday.io/project/182527/instructions).

## Panel config

After the boot or first restart, please enter via serial console or terminal and please configure your settings, at least one WiFi network and 3 currencies. Type `help` and you should able to have the next menu:

![CPanel CLI demo](https://raw.githubusercontent.com/hpsaturn/crypto-panel/refs/heads/master/images/cli_help.jpg)

Please visit the [project page](https://hackaday.io/project/182527-crypto-news-eink-panel) and its full instruccions and steps on the [Hackaday Instructions Section](https://hackaday.io/project/182527/instructions) for complete details.

## Firmware build

Please install first [PlatformIO](http://platformio.org/) open source ecosystem for IoT development compatible with **Arduino** IDE and its command line tools (Windows, MacOs and Linux). Also, you may need to install [git](http://git-scm.com/) in your system.

```bash
pio run --target upload
```

## Demo

!!**This demo is old but quiet close to the new CLI experience**!!  

[![Crypto panel](https://raw.githubusercontent.com/hpsaturn/esp32-wifi-cli/master/images/cryptopanel_preview.jpg)](https://youtu.be/oyav6SvN870)  
(deprecated)

## Credits

- This project use the [nmcli project](https://github.com/hpsaturn/esp32-wifi-cli/) and [Shellminator](https://github.com/dani007200964/Shellminator)
- Thanks to @techiesms by first idea and [original project](https://github.com/techiesms/) for Arduino IDE
- Thanks to @hacksics from some icons and fonts of project [HA dashboard project](https://github.com/hacksics/lilygo-t5-47-ha)
- This project use the last version of [EPDIY driver](https://github.com/vroland/epdiy) of @vroland
