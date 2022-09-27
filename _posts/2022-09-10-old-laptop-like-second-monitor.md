---
layout: post
title:  "Your old laptop like a second monitor"
date:   2022-09-10
excerpt: "How to connect old monitor to virtual xorg space"
feature: http://hpsaturn.com/assets/img/xorg-virtual-monitor.png
tag:
- Debian
- Xorg
comments: false
---

# Your old laptop like a second monitor

I have a Asus Zenbook UX534, it isn't a old laptop but I wanted join it to my new desktop machine, because this laptop has two monitors. For this reason here I post the conclusions of this goal.

I don't want repeat some nice guides that I found, better I'm going to reference here:

## Context

Previous documentation:

[Extreme Multihead - Extending a desktop beyond](https://wiki.archlinux.org/title/Extreme_Multihead#Extending_a_desktop_beyond_the_local_system)
[Multiple dummy monitors on linux](https://unix.stackexchange.com/questions/651714/multiple-dummy-monitors-on-remote-headless-linux-for-vnc-to-local-multiple-monit)


## Results

After this documentatioo, I wrote a simple utility that enable virtual spaces in my current xorg session on my desktop, and launch the VNC server for my laptop, than the laptop will start a vnc client and show what happend in this virtual desktop. Of course you are able to put many monitors if you want too.

### External monitor alternative

```python
#!/bin/bash

# Secondary display resolution
W=1920       # Virtual display width
H=1080       # Virtual display height

# Primary display config
P="HDMI-A-0"        #is the main screen, it can be calle eDP-1 or eDP1 depending on the driver
O="DisplayPort-0"   #can be a virtual (recommended if possible) or real output accepted by the xog driver.
                    #more info: https://wiki.archlinux.org/title/Extreme_Multihead#VNC

############# END SETUP ##############################################################
PW=`xrandr --current | egrep '\*' | awk '{print $1;}' | cut -d x -f 1 | head -n 1`

# Create the virtual display
gtf $W $H 120 | sed '3q;d' | sed 's/Modeline//g' | xargs xrandr --newmode
gtf $W $H 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --addmode $O
gtf $W $H 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --output $O --right-of $P --mode
sleep 1

# Android device init
# Forward the VNC port to your device and start a VNC session
# adb reverse tcp:5900 tcp:5900 # uncomment for Android secondary display
echo "Starting VNC server"
x11vnc -display :0 -clip ${W}x${H}+${PW}+0 -repeat -forever -nonap -multiptr -noxfixes

# When cancel this script, turn off the virtual display
xrandr --output ${O} --off

# On the old laptop please run:
# vncviewer -shared -ViewOnly -Fullscreen <host ip>
```
[source code](https://gist.github.com/hpsaturn/7b6d15f149eb5bb9bdb19b94b1b34c42)

### Two external screens

The next version has a some improvements to stat/stop/status the virtual spaces and servers on **WindowMaker** window manager:

```python
#!/bin/bash

# Secondary display resolution
Wrt=1920       # Virtual display width (top right)
Hrt=1080       # Virtual display height
Wrb=2160       # Virtual display width (bottom right)
Hrb=1080       # Virtual display height

# Primary display config
P="HDMI-A-0"        #is the main screen, it can be calle eDP-1 or eDP1 depending on the driver
Ort="DisplayPort-0" #can be a virtual (recommended if possible) or real output accepted by the xog driver.
Orb="HDMI-A-1"      #can be a virtual (recommended if possible) or real output accepted by the xog driver.
                    #more info: https://wiki.archlinux.org/title/Extreme_Multihead#VNC
                    #Virtual device using evdi driver:
                    #sudo modprobe evdi initial_device_count=1
                    #xrandr --setprovideroutputsource 1 0

############# END SETUP ##############################################################

showHelp() {
  printf "\r\nusage:\t[start|pause|stop|status|help]\r\n\n"
  printf "start: \tstart xrandr config and vnc servers\r\n"
  printf "pause: \tdisable xrandr virtual screens\r\n"
  printf "stop : \tdisable screens and stop vnc servers\r\n"
  printf "status:\txrandr and vnc servers status\r\n\n"
}

configScreens () {
  # Principal screen width
  PW=`xrandr --current | egrep '\*' | awk '{print $1;}' | cut -d x -f 1 | head -n 1`
  # Create the virtual display
  gtf $Wrt $Hrt 120 | sed '3q;d' | sed 's/Modeline//g' | xargs xrandr --newmode
  gtf $Wrt $Hrt 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --addmode $Ort
  gtf $Wrt $Hrt 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --output $Ort --right-of $P --mode
  sleep 1
  gtf $Wrb $Hrb 120 | sed '3q;d' | sed 's/Modeline//g' | xargs xrandr --newmode
  gtf $Wrb $Hrb 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --addmode $Orb
  gtf $Wrb $Hrb 120 | sed '3q;d' | sed 's/Modeline//g' | awk '{print $1;}' | sed 's/^.\(.*\).$/\1/' | xargs xrandr --output $Orb --below $Ort --mode
  sleep 1
  kill -usr1 `pgrep wmaker | tail -1` # restarting window manager for it recognize the new setup
  sleep 0.5
}

pauseScreens () {
  xrandr --output ${Ort} --off
  sleep 1
  xrandr --output ${Orb} --off
}

startVncServers () {
  # Android device init
  # Forward the VNC port to your device and start a VNC session
  # adb reverse tcp:5900 tcp:5900 # uncomment for Android secondary display
  echo "Starting VNC server"
  x11vnc -bg -display :0 -clip ${Wrt}x${Hrt}+${PW}+0 -rfbport 5900 -repeat -forever -nonap -multiptr -noxfixes
  sleep 1
  x11vnc -bg -display :0 -clip ${Wrb}x${Hrb}+${PW}+${Hrt} -rfbport 5901 -repeat -forever -nonap -multiptr -noxfixes
  #
  # On the old laptop please run:
  # vncviewer -shared -ViewOnly -Fullscreen <host ip>
}

stopVncServers () {
  x11vnc -R stop
  x11vnc -clear-all
}

screenStatus () {
  printf "\r\nMain display and virtual screens:\r\n\n"
  xrandr --display :0 | grep connected | awk '{print $1}'
  printf "\r\nX11 VNC servers running on:\r\n\n"
  ps aux | grep x11vnc | grep rfbport | awk '{print $18}'
}


if [ "$1" = "" ]; then
  showHelp
else
    case "$1" in
        start)
            configScreens
            startVncServers
            ;;
        pause)
            pauseScreens
            ;;
        stop)
            pauseScreens
            stopVncServers
            ;;
        status)
            screenStatus
            ;;
        help)
            showHelp
            ;;
        *)
            showHelp
            ;;
    esac
fi
```

[source code](https://github.com/hpsaturn/linux_scripts/blob/master/virtual_screens)

## Troubleshooting

The main issue or concern is that in some Linux versions with some GPU drivers like AMD or Nvidia, the `virtual displays` are not supported and you need use some hacks on XOrg or use **dummy** drivers, that is mentioned in the references. But on the other hand you can use **unused outputs** in your system, for example in my scripts I used the un used `DisplayPort` (for Mac) and the second unused `HDMI` on my desktop machine.