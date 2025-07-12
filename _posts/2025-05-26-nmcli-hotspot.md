---
layout: post
title:  "Hotspot using Network Manager CLI"
date:   2025-05-26
excerpt: "How to share your connection using NMCLI on headless devices"
feature: "http://influxdb.canair.io:8080/images/pilauncher_feature.jpg"
tag:
- GNU-Linux
- Debian
- RaspberryPi
comments: false
---

## NMCLI Hotspot

This guide is an update of my old guide for creating a Hotspot using legacy Debian commands. The `network-manager` package (via the `nmcli` command) simplifies the setup and reduces system overhead, making it ideal for headless devices like servers or RaspberryPi.

## Prerequisites

- A system with **two network interfaces** (e.g., Ethernet `eth0` and Wi-Fi `wlan0`).
- The `network-manager` package installed (provides `nmcli` command).
- Ensure Wi-Fi supports "AP mode":

```bash
iw list | grep "Supported interface modes" -A 8 | grep "AP"
```

### Hotspot Creation

```bash
nmcli con add type wifi ifname wlan0 con-name Hostspot autoconnect yes ssid Hostspot
nmcli con modify Hostspot 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared
nmcli con modify Hostspot wifi-sec.key-mgmt wpa-psk
nmcli con modify Hostspot wifi-sec.psk "veryveryhardpassword1234"
nmcli con up Hostspot
```

Notes:

- Replace MyHotspot with your desired SSID.
- Use bg band for 2.4 GHz or a for 5 GHz (check adapter support).

**Done!** you are able to connect to your Hotspot. And also you could up/down the hotspot connection in any moment using the last command line only. You don't need repeat the previous steps.

#### Verify Hotspot Subnet

```shell
nmcli con show Hotspot | grep ADDRESS
```

### Firewall Considerations

If you have a firewall in your system, the network-manager is shared IP method auto-configures NAT, but maybe your firewall might block traffic. The next are some rules for some services that you could need:

#### For ufw:

```shell
sudo ufw allow in on wlan0 proto udp port 67  # DHCP
sudo ufw allow in on wlan0 proto udp port 53  # DNS
sudo ufw route allow in on wlan0 out on eth0  # Forwarding
```

#### For iptables:

```shell
iptables -t nat -A POSTROUTING -s 10.42.0.0/24 -o eth0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

### Norvpn Addon

If you also needs that you hotspot runs under a VPN, you can install for instance [NordVPN](https://support.nordvpn.com/hc/en-us/articles/20226600447633-How-to-log-in-to-NordVPN-on-Linux-devices-without-a-GUI) that it has commmand line capability.

After the nordvpn installation, you also should need add somo ports, like this:

```bash
nordvpn whitelist add port 22
nordvpn whitelist add port 67
```

Also you should permit traffic from the Hotspot subnet, for instance:

```bash
nordvpn add subnet 10.42.0.0/24
```

### RaspberryPi Launcher Addon

If you wants command the on/off of your hotspot and also choose between different VPN cities or configs, you also could considerate add [RaspberryPi Launcher](https://github.com/hpsaturn/pilauncher/tree/master?tab=readme-ov-file#readme)
app for your RaspberryPi.

[![Youtube vide demo PiLauncher](https://raw.githubusercontent.com/hpsaturn/pilauncher/master/screenshots/demo_youtube.jpg)](https://youtu.be/iNSw1nZpOEk?si=aX4mq4WVJhwrCm_X)
