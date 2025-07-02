---
layout: post
title:  "uConsole - x48ng"
date:   2025-07-03
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

### Transfer files

Maybe the easy way for transfer files to the emulator is using Kermit protocol, but is possible also other protocols like XModem. Here I'm going to explain the alternive using **ckermit**:

```shell
sudo apt install ckermit
```

Then, write a script in you user binary directory on ~/bin for instance with the name kermit48, with the next contents:

```bash
#!/bin/bash

if [ "$1" = "" ]; then
  echo "usage: kermit48 /dev/ttyxx"
  exit 1
fi

kermit -l $1 -C "set modem type direct, set prefixing all, set speed 9600, set carrier-watch off, set flow none, set parity none, set block 3"
```

(don't forget set executable permissions with `chmod +x ~/bin/kermit48`)

Run the emulator, and its bottom line you are able to see something like this:

```shell
wire: /dev/pts/16
```

Then run kermit script like this:

```shell
kermit48 /dev/pts/16
```

On the kermit terminal, you are able to navitage to any directory and send commands with `send xxx`. Previosly you should start in the emulator, the kermit server with `server` command or with the RIGHT SHIFT + RIGHT ARROW.

![x48ng uconsole kermit demo](/assets/img/uconsole_x48ng_kermit_demo.jpg)

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