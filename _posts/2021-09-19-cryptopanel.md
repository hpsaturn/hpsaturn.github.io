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

## References

`Sourcecode`: last version of this guide and source code in [GitHub](https://github.com/hpsaturn/crypto-panel) <br/>
`Webinstaller`: this project has its own web installer [here](https://hpsaturn.com/crypto-panel-installer/) <br/>
`PlatformIO`: is an open source ecosystem for [IoT development](https://platformio.org/) <br/>

## Features

- Panel installation and configuration using an easy [Web installer](https://hpsaturn.com/crypto-panel-installer/)
- Configuration via command line (CLI) using the web installer or any Serial console
- Support hundreds of coins from [Coingecko API](https://api.coingecko.com/api/v3/coins/list?include_platform=false)
- Random cryptocurrencies news from Cointelegrah and others news portals
- QR Code of each coin news link
- Base currency USD/EUR configurable
- Firmware update via automatic OTA updates
- Deep sleep configurable (default: 30 min).
- Panel temperature ambient configurable (for improve colors)
- Time Zone configurable
- Included optional basic 3D-Print frame

![CryptoPanel dashboard photo](https://hpsaturn.com/assets/img/crypto_panel_preview.jpg)

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
