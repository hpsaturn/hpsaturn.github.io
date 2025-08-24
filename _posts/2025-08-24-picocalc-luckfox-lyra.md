---
layout: post
title:  "Picocal Luckfox Lyra"
date:   2025-08-24
excerpt: "Picocalc with Linux from scratch without use Windows tools"
feature: "https://hpsaturn.com/assets/img/uconsole_x48ng_preview.jpg"
tag:
- GNU-Linux
- Picocalc
- Hp48
comments: false
---

## Picocal Luckfox Lyra

Here's the current status of my migration of the x48ng emulator to Picocalc using a modified Luckfox Lyra board and its Linux from scratch modified. For now, only the Ncurses version works, but I plan to add SDL support as well. It's already functional! Also I did it entirely without closed-source Windows tools, which was the biggest challenge! :)

Current issues and steps to replicate it:
[here](https://github.com/benklop/picocalc-luckfox-lyra/pull/7)

## Pull Request status

This PR is trying to add [x48ng emulator](https://github.com/gwenhael-le-moine/x48ng) package to the buildroot system.

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
