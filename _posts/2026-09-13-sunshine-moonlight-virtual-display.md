---
layout: post
title:  "Virtual display using Sunshine"
date:   2026-09-13
excerpt: "How to use unused screens like a real displays for your desktop"
tag:
- GNU-Linux
- Debian
- Gaming
- Xorg
comments: false
---

Some years ago I wrote a [guide](https://hpsaturn.com/old-laptop-like-second-monitor/) that explained how to give support to old hardware, like a tables or old laptops and join them like a real displays connected to your main PC, but it has a low performance and some issues because old and vncserver was unmaintained.

But now we have [Sunshine](https://app.lizardbyte.dev/Sunshine/), a recommended game streaming server, very optimized to launch your virtual screens using [Virtual Displaylink](https://github.com/AdnanHodzic/displaylink-debian) to have the same idea.

This guide only wants put some tips for Linux using xorg to do that, because I now that Sunshine have many source posibilities and we have different GPU boards, for that this guide is not for all scenarios.

## Scenario

- You have a GNU-Linux without Nvidia GPU (with Nividia is possible don't use Displaylink)
- You have unused screens, for instance an Android Tablet or old laptop.
- You want have more screens, not play games (but it is possible :D)

## Virtual Screens

First, we need to add virtual screens to our X session, for instance a virtual monitor below of main monitor that it will be our Android tablet, and maybe a second screen that will be our old tablet. For that we need to generate virtual screen devices and configure them. For that we are going to use [Virtual Displaylink](https://github.com/AdnanHodzic/displaylink-debian). Please see the documentation and install it. At the end you should have something like that:

```bash
DVI-I-4-4 disconnected (normal left inverted right x axis y axis)
DVI-I-3-3 disconnected (normal left inverted right x axis y axis)
DVI-I-2-2 disconnected (normal left inverted right x axis y axis) 
DVI-I-1-1 disconnected (normal left inverted right x axis y axis)
```

## Virtual Screen Config

Now, you have 4 monitors more in theory, but you need to add this to your setup, for instance using `xrandr` command. Here, for instance this script could configure my Tablet space:

```bash
#!/bin/bash

MAIN_HDMI=HDMI-A-0
DVI_OUTPUT="DVI-I-1-1"
MODE="1920x1080_60.00"

config_virtual () {
  # Check if the mode already exists in the current X server instance
  if ! xrandr | grep -q "${MODE} (0x"; then
  ¦ xrandr --newmode ${MODE} 173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
  ¦ sleep 0.5
  fi
  xrandr --addmode ${DVI_OUTPUT} ${MODE} 2>/dev/null
  sleep 0.5
}

# this put the tablet screen below of my main screen: 
enable_virtual () {
  xrandr --output $DVI_OUTPUT --mode ${MODE} --below ${MAIN_HDMI}
  sleep 1
}

# your desktop manager restart. Here for WMaker
wmaker_restart () {
  kill -usr1 `pgrep wmaker | tail -1`
}
```

After that, you could use `arandr` command to see your final setup, for instance for me:

{% capture images %}
  {{ '/assets/img/sunshine_arandr_sample.jpg' | relative_url }}
{% endcapture %}
{% include gallery images=images cols=1 %}

## Sunshine server

[Sunshine](https://app.lizardbyte.dev/Sunshine/) has many options and possibilities, but it is easy to configure. But some things are important here:

- You will need to able to change or open some firewall ports
- Depends if you have NVidia or AMD.
- Is important first configure the virtual space before to launch Sunshine
- In theory works with Wayland. (for me it still has many issues)
- Please try first with the default config of Sunshine

My current config:

```python
adapter_name = /dev/dri/renderD128
capture = x11
controller = disabled
encoder = vaapi
keyboard = disabled
output_name = 5
port = 48989
sw_preset = ultrafast
sw_tune = stillimage
vaapi_strict_rc_buffer = enabled
```

In `~/.config/sunshine/sunshine.conf`:

Tips:

- `capture = x11` for me fixed many issues, but maybe is different for you.
- Please enter to network setting of Sunshine to understand which ports more you need in your firewall.
- Output name seems that is the list number position of your xrandr output, but maybe you will need to play with that.

## Moonlight

Moonlight is the client of Sunshine server. It is able to run in many architectures. Add the host IP, type the PIN shown in the Sunshine web UI and match the client resolution (1920x1080). You don't need many changes in Moonlight config, maybe the bitrate. I limited it to 2Mb, and is enough for full HD.
