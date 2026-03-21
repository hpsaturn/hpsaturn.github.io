---
layout: post
title:  "OpenStreetMap tiles for eInk displays"
date:   2026-03-21
excerpt: "Alternative to pay APIs for get scale grays map tiles optimized for eInk displays"
feature: "https://hpsaturn.com/assets/img/picocalc_x48ng_preview.jpg"
tag:
- GNU-Linux
- Picocalc
- Hp48
comments: false
---

## Map tiles for eInk displays

On some map apps using microcontrollers and eInk displays, like Meshtastic or Meshcore, you need special map tiles, but some guides using restricted API to download maps with special rules for improve the tiles for this kind of screens, the problem is that in some cases these APIs have a free quote or don't have all zones.

This alternative use Maperitive without tricky rules, only the default rule and a some scripts to post procesing these tiles. That means that this guide also works with old downloaded tiles for TFT or color screens.

Also in this guide, I included some tips for Linux, because this tool only works there, and some improvements in the copy of these tiles to the SDCard of the device, in this case all here was tested on a LilyGO T-Deck Pro that has eInk display.

## Maperitive installation (Linux)

Current issues and steps to replicate it:
[here](https://github.com/benklop/picocalc-luckfox-lyra/pull/7)

## Migration status

- [x] Compiling issues
- [x] Linking issues
- [x] Package installation issue
- [x] execution test: x48ng -help [PASS]
- [x] execution full emulation [PASS]
- [x] F7 shortcut fails (general problem with Lyra). Fix: F1 new Exit
- [ ] SDL2 support
- [ ] hash commit missing for buildroot
- [ ] possible patch for improve buildroot recipe

[![Image](https://github.com/user-attachments/assets/c619a56a-a758-4fa0-b04e-9f73c8ae40c2)](https://youtu.be/7uAfvPzB3bs)

## Issues

- [x] https://github.com/gwenhael-le-moine/x48ng/issues/29
- [x] https://github.com/gwenhael-le-moine/x48ng/issues/30
- [x] https://github.com/gwenhael-le-moine/x48ng/issues/31
- [x] https://github.com/gwenhael-le-moine/x48ng/issues/33
- [ ] https://github.com/gwenhael-le-moine/x48ng/issues/35

## Installation steps

### buildroot base

```bash
git clone -b x48ng_package git@github.com:hpsaturn/picocalc-luckfox-lyra.git
cd picocalc-luckfox-lyra
./setup.sh
./build.all
```

### x48ng build and install

```bash
./build.sh buildroot-make:x48ng-dirclean
./build.sh buildroot-make:x48ng
./build.sh buildroot-make:fbterm
./build.sh all
./flash.sh update
```

### Lyra execution steps

1. run x48ng in your host. 
2. transfer your `.config/x48ng` host directory to your Lyra home directory
3. login in your Lyra for instance via adb
4. change the fbterm config file with mono font and size 7
5. run `x48ng --tui-tiny --mono` (try with -r if not response and press F5 to ON)
6. F1 to exit

![preview x48ng on Picocalc](https://hpsaturn.com/assets/img/picocalc_x48ng_preview.jpg)