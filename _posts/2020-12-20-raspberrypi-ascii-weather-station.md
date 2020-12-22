---
layout: post
title:  "RaspberryPi ASCII Weather Station"
date:   2020-12-20
excerpt: "WiFi Weather Station on Golang without X Desktop. (xinit alternative)"
feature: http://hpsaturn.com/assets/img/ascii_weather_feature.jpg
tag:
- RaspberryPi
- Debian
- Raspbian
comments: false
---

# RaspberryPi ASCII Weather Station

The next guide reach that RaspberryPi launch a Go program that show the current weather and 3 days of weather forecast. I'm using a fork that I did from [wego](https://github.com/hpsaturn/wego) developed by [Schachmat](https://github.com/schachmat). The main program is the only program that run in the Pi, for that I forced it to start with xinit on full screen, also I included some improvments for:

- [x] No window manger (only xinit with fullscreen mod)
- [x] disabled extra system services
- [x] improved system logs only in memory
- [x] disabled some devices for improve power consumption
- [x] fast boot improvements


{% capture images %}
  {{ site.url }}/assets/img/ascii_weather_station.jpg
{% endcapture %}
{% include gallery images=images cols=1 %}


# Materials

- [x] RaspberryPi (any version should be works)
- [x] Any TFT display compatible (I'm using a [4inch HDMI LCD](https://www.waveshare.com/wiki/4inch_HDMI_LCD_(H)))

Note: you don't need keyboard or mouse, this guide is a **headless** installation, we only need SSH.

# Prerequisites

## Open Weather Map API key

You can create an account and get a free API key by [signing up](https://home.openweathermap.org/users/sign_up)

## RaspiOS config on PC

Please first install in your SD [RaspiOs lite](https://www.raspberrypi.org/software/operating-systems/) or any Raspberry OS compatible with it on your RaspberryPi. Before run it, please add the next changes:

In the **boot** partition or boot folder, modify and add the next configs:

### Enable SSH:

Add a simple empty file for enable SSH access in boot folder:

```bash
touch ssh
``` 

### WiFi Credentials

Add your WiFi secrets on a new file **wpa_supplicant.conf** in the boot folder, you can generate it like this:

```bash
wpa_passphrase yourwifiname yourpassword > wpa_supplicant.conf
```

edit it and put the next line at the start of file:

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
``` 

More info about WiFi headless configure [here](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md).


# Connect your RaspberryPi 

## Dependencies

Via [SSH](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md), please update and install the next packages for build and run the wego application:

```bash
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install --no-install-recommends xserver-xorg xserver-xorg-legacy xinit xterm golang build-essential git
```

Config that Pi user can launch the xserver without root:

```bash
sudo dpkg-reconfigure xserver-xorg-legacy
```

in the **console menu** please select **Anybody**

## Build and install Wego

```bash
go get -u github.com/hpsaturn/wego
```

Then **configure wego** like is described [here](https://github.com/hpsaturn/wego?organization=hpsaturn&organization=hpsaturn#setup). Remember only fill the settings for **openweathermap**.

**NOTE**: Unfortunately forecast.io, now Darksky, was acquired by the fuck company Apple, and they closed it, for this reason please only use these API, the **steps 2 and following**.


## Launcher

Please edit **/etc/rc.local** and put the next line before the last line:

```bash
sudo su -l pi -c "xinit -geometry =800x400+0+0 -fn 8x13 -j -fg white -bg black /home/pi/go/bin/wego -- -nocursor" &
```

reboot the Pi and it is all, the Pi will start the Wego app like unique X application.


{% capture images %}
  {{ site.url }}/assets/img/ascii_weather_station.gif
{% endcapture %}
{% include gallery images=images cols=1 %}


---

# Improvements

## Speed: Boot mods

In **config.txt** add the next lines to end of file:

```yml
[all]
arm_freq_min=250
core_freq_min=100
sdram_freq_min=150
over_voltage_min=0
disable_splash=1
dtoverlay=pi3-disable-bt
dtparam=act_led_trigger=none
dtparam=act_led_activelow=on
boot_delay=0
```
With that, we have: reduced CPU usage, no boot splash, disabled bluetooth, disabled LEDs and reduced the boot delay. 


In **cmdline.txt**:

Edit and add to the end of the line the next changes, **quiet** for remove boot verbose, **fastboot**, **noswap** and **ro**, for improve SD life and improve boot speed:

```bash
console=serial0,115200 console=tty1 root=PARTUUID=132a822c-02 rootfstype=ext4
elevator=deadline fsck.repair=yes quiet rootwait fastboot noswap ro
```

## Speed: Disable some services

Optional, you could disabled some unused services, for example:

```bash
sudo systemctl disable dphys-swapfile.service
sudo systemctl disable keyboard-setup.service
sudo systemctl disable apt-daily.service
sudo systemctl disable raspi-config.service
sudo systemctl disable triggerhappy.service
sudo systemctl disable avahi-daemon.service
```

More info about this improvement [here](http://himeshp.blogspot.com/2018/08/fast-boot-with-raspberry-pi.html).


## Speed and life time of SD

You also could disabled some write process for improve speed and SD life:

Remove and disable the next services:

```bash
sudo apt-get remove --purge wolfram-engine triggerhappy anacron logrotate dphys-swapfile
```

```bash
sudo systemctl disable bootlogs
sudo systemctl disable console-setup
```

Replace your log manager, removing the standard syslog output of log files to /var/log and instead replace it with the busybox in-memory logger:

```bash
sudo apt-get install busybox-syslogd
sudo dpkg --purge rsyslog
```

## Power consumption

I found a [excellent guide](https://learn.pi-supply.com/make/how-to-save-power-on-your-raspberry-pi/) about that, the more important thing is turn off the USB ports, but there they described other improvements.

For example for turn off USB ports, put the next line **before** xinit line on **/etc/rc.local**:

```bash
echo '1-1' |sudo tee /sys/bus/usb/drivers/usb/unbind &
```

## Troubleshooting

Any feedback, bug or issue, please report it in the [issue section](https://github.com/hpsaturn/hpsaturn.github.io/issues).