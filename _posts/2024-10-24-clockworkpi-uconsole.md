---
layout: post
title:  "uConsole - x48ng"
date:   2024-10-24
excerpt: "How to use the uConsole like a Hp48 calculator"
feature: "https://hpsaturn.com/assets/img/uconsole_x48ng_preview.jpg"
tag:
- GNU-Linux
- Debian
- RaspberryPi
- Hp48
comments: false
---

## x48ng - uConsole

Diving into the past with this nostalgic HP48Gx, a powerful handheld calculator from the 90s, now running here on an x48ng emulator in this #uConsole or any GNU-Linux device.

## Installation

Dependencies:

```bash
sudo apt install pkgconf libreadline-dev libsdl2-dev libx11-dev libxext-dev liblua5.4-dev git build-essential
```

From the official [x48ng Github](https://github.com/gwenhael-le-moine/x48ng) copy the repo url, clone it and perform the next commands:

```bash
git clone https://github.com/gwenhael-le-moine/x48ng
cd x48ng &&  make clean-all && make
sudo make install PREFIX=/usr
```

## Usage

x48ng has many options and GUI runners alternatives. But, for me, the best for the uConsole and maybe for many terminal lovers, is the next:

```bash
x48ng --tui-tiny --mono -T
```

This alternative runs a ncurses version for any terminal:

![x48ng Filer Screenshot](/assets/img/uconsole_x48ng_filer.jpg)

### ROM state backups

For now the only way to do backups of the state of the emulator is saving the working directory files of the x48ng, like this:

```shell
cd ~/.config/x48ng/
tar zcf ~/bkp`date +%Y%m%d_%H%M%S`.tar.gz port1 port2 ram state config.lua
```

and restore any state, with something like this:

```shell
tar zxf ~/bkp20241020_135648.tar.gz -C ~/.config/x48ng/
```

## Demo

It's a little demo, of how to we used in these years the Jazz 6.8 library and TED editor to navigate, edit, and even decompile system calls and third-party programs written in Assembler or SysRPL, right on your device, and do all that in the middle of your coding session, was truly amazing in those years. Thanks to these awesome programs and features, we had a lot of "open source" software in this era.

[![x48ng uConsole Demo](/assets/img/uconsole_x48ng_video.jpg)](https://www.youtube.com/shorts/AzAMqLjMT-Q)