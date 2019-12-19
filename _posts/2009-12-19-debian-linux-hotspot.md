---
layout: post
title:  "Easy Debian hotspot loader"
date:   2009-12-19
excerpt: "Debian Hotspot script for loading WPA AP in Wlan0"
feature: ""
tag:
- GNU-Linux
- Debian
- Bash
comments: false
---

# Debian Hotspot loader

#### Last update: 2019.12.20

Debian hotspot on Wifi device using `dnsmask` and `hostapd` with a simple bash script for `enable/disable` it.

## Installation

Please create the next script in `/usr/bin/loadhotspot`, also you can download it [here](https://raw.githubusercontent.com/hpsaturn/linux_scripts/master/loadhotspot)

``` bash
#!/bin/sh
###################################################################
# Load hotspot
# 2009-2010 Hpsaturn v1.0
# hpsaturn@gmail.com
###################################################################

DEVICE=wlo1
MODULE=iwldvm

start_spot () {
    remove_module
    sleep 1
    install_module
    sleep 2
    ifconfig $DEVICE down
    sleep 1
    echo -n "set static gateway.."
    ifconfig $DEVICE 192.168.3.1 netmask 255.255.255.0
    ifconfig $DEVICE up
    echo "ok" 
    echo "start hostapd:"
    service hostapd start
    echo "start NAT:"
    #/etc/init.d/load_iptables restart
    sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
    iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -d 0.0.0.0/0 -j MASQUERADE
    echo "start dnsmask:"
    cp /etc/dnsmasq_ap.conf /etc/dnsmasq.conf
    service dnsmasq restart
}

stop_spot () {
    stop_services
}

stop_services () {
    echo -n "stop services.."
    ifconfig $DEVICE down
    service hostapd stop
    service dnsmasq stop
    echo "done."
    remove_module
}

remove_module () {
    echo -n "remove modules.."
    modprobe -rv $MODULE
    echo "done."
}

install_module () {
    echo -n "install modules.."
    modprobe $MODULE
    echo "done."
}

case "$1" in
  start)
    start_spot 
    ;;
  stop)
    stop_spot
    ;;
  restart)
    stop_spot
    start_spot
    ;;
  *)
    echo "Usage: load_hotspot {start|stop|restart}"
    exit 1
    ;;
esac

exit 0

```

Please change the next lines to right values

``` bash
DEVICE=wlo1      # Change to right Wlan0 device
MODULE=iwldvm    # Change for right Kernel module device
```

### Dependencies

Install the next packages:

``` bash
sudo apt-get install hostapd dnsmasq
```

## Configuration

Please add `/etc/hostapd.conf`

``` bash
interface=wlo1      # The same Wifi device than above
ssid=HostapName     # SSID for your network
channel=1
hw_mode=g
auth_algs=1
wpa=3
wpa_passphrase=xxyy        # network password
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
rsn_pairwise=CCMP
#ignore_broadcast_ssid=1   # uncomment it if you want SSID hide
```

Add dnsmasq config file `/etc/dnsmasq_ap.conf` like this:

``` bash
interface=wlo1                              # the same than above
listen-address=127.0.0.1                   
bind-interfaces
domain=YourDomainOrAny                      # set network domain name
dhcp-range=192.168.3.100,192.168.3.200,12h  # define dhcp range
```

## Starting hotspot deamons

``` bash
sudo loadhotspot start
```

Output:  

```
remove modules..done.
install modules..done.
set static gateway..ok
start hostapd:
start NAT:
start dnsmask:
```

## Stop hotspot

``` bash
sudo loadhotspot stop
```

Output:  

```
stop services..done.
remove modules..rmmod iwldvm
done.
```

### Troubleshooting

If you have the next error:

``` bash
Failed to start hostapd.service: Unit hostapd.service is masked.
```

Please run:

``` bash
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```
