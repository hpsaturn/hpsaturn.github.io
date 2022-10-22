---
layout: post
title:  "Virtual Serial Port"
date:   2022-10-08
excerpt: "How to create a virtual port like /dev/tty over WiFi"
feature: http://hpsaturn.com/assets/img/virtual_serial_port.jpg
tag:
- C++
- IoT
- GNU-Linux
- OpenHardware
- Arduino
- GPS
comments: false
---

# Virtual Serial Port

**W A R N N I N G**
The current guide is in development.

Working with IoT devices sometimes you need know what happend in your application but you don't have a connection to the device via serial port. Also sometimes you need connect it to other software but the IoT device maybe is in outdoors. I show you a simple alternative to export the Serial Port via Wifi and reproduce it on a fake device for using in your OS, and also a practical application using a remote GPS connected to PyGPSClient.

Limitations:

- Read only (for now)
- Possible conflicts with WiFi IoT apps. It use UDP.
- The current implementation only works in Linux

## Firmware

We need redirect the Serial output via UDP. For now I found some implementation that do that, but the idea is write a library.

In our example, we need redirect the Serial2 output where is connected our GPS to UDP socket, but you could redirect your debug messages too.

```cpp
HardwareSerial *gps;

bool udp_write(uint8_t text, int port) {
  if (WiFi.status() == WL_CONNECTED) {
    Udp.stop();
    Udp.begin(WiFi.localIP(), 3332);
    delay(10);
    if (Udp.beginPacket(udpip, port)) {
      Udp.write(text);
      Udp.endPacket();
      return (true);
    } else {
      Udp.stop();
      return (false);
    }
  } else {
    return (false);
  }
}

void loop() {
  ...
  udp_write(gps->read(), 9000); // it take the GPS characters
                                // from UART (Serial2 in the ESP32)
                                // and send it to the PC in 9000 port
  ...
}
```

The full example use the [ESP32 WiFi CLI library](https://github.com/hpsaturn/esp32-wifi-cli#readme) for configure the WiFI credentials, and also to choose the kind of the redirection, to UDP or to the primary serial port. More details here:  
[GPS UDP modem - source code](https://github.com/hpsaturn/esp32-wifi-cli/tree/master/examples/gps-udp-modem)

## Virtual Interface

We need first open the port in your system, for example with `ufw`:

```bash
sudo ufw allow 9000/udp
```

For testing your UDP connection, launch the firmware, then review the messages with netcat:

```bash
netcat -u -l -p 9000
``` 

Output:

```bash
avp:~ȹ netcat -u -l -p 9000 
,,,,,,,,N*30
$GPGGA,,,,,,0,00,99.99,,,,,,*48
$GPGSA,A,1,,,,,,,,,,,,,99.99,99.99,99.99*30
$GPGSV,1,1,00*79
$GPGLL,,,,,,V,N*64
$GPGLL,,,,,,V,N*64
$GPRMC,,V,,,,,,,,,,N*53
$GPGGA,,,,,,0,00,99.99,,,,,,*48
$GPGSA,A,1,,,,,,,,,,,,,99.99,99.99
``` 
Now we need redirect this output to fake serial port. For that we need first launch in one terminal the input and output sockets, for that we using the next command:

```bash
socat -d -d pty,raw,echo=0 pty,raw,echo=0
```

Output:

```bash
2022/10/08 01:20:35 socat[1311338] N PTY is /dev/pts/14
2022/10/08 01:20:35 socat[1311338] N PTY is /dev/pts/15
2022/10/08 01:20:35 socat[1311338] N starting data transfer loop with FDs [5,5] and [7,7]
```
The first one is the output Serial port, and the second one is the input.  

In other terminal connect the UDP port to the fake input serial port:

```bash
netcat -u -l -p 9000 > /dev/pts/15
```

And that it's! You can test the serial port using `tail`:

```bash
tail -f /dev/pts/14
```

With that you able to use the serial, for example for debugging or some application like `PyGPSClient`.

## Python GPS Client

![PyGPSClient](/assets/img/pygpsclient.jpg)

For instance we can use our virtual port with this soffware. For that you only put the new serial port in the next [python script](https://github.com/semuconsulting/PyGPSClient/blob/master/examples/socket_server.py) like this: 

Change the last lines to:

```python
  SERIAL = "/dev/tty.usbmodem14101"
  BAUD = 115200
  PORT = 50010
  MAXCLIENTS = 5
```

launch the python socket server:

```python
python3 socket_server.py
```

Then launch PyGPSClient and configure a TCP/UDP connection like this:

![PyGPSClient UDP config](/assets/img/pygpsclient_udp_config.jpg)

Click on TCP/UDP button and with that your remote GPS device will be connect :D




