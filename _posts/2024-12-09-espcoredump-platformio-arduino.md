---
layout: post
title:  "ESPCoredump with Arduino Framework"
date:   2024-12-09
excerpt: "Using esp-coredump with Arduino Framework and PlatformIO"
feature: http://hpsaturn.com/assets/img/esp-coredump-preview.jpg
tag:
- PlatformIO
- Arduino
- IDF
- C++
- GNU-Linux
- IoT
comments: false
---

# ESP Coredump with Arduino Framework using PlatformIO

The latest version of PlatformIO and Espressif for the Arduino Framework includes the option for Coredump in FLASH. This means that you can capture exceptions with more information even when you are not connected to the device. The coredump binaries will be stored in your flash memory, allowing you to retrieve and analyze this information later. Additionally, this coredump provides more information than the standard exception decoder filter in PlatformIO, such as the complete stack backtrace, the current tasks at the moment of the crash, and other relevant details. Previously, this option was only available for IDF projects [^1], but it is now possible with the Arduino Framework. This guide aims to explain what you need to get started.

## Prerequisites

I assume that you are familiar with PlatformIO and have already deployed any firmware or app using the Arduino framework option, and maybe you will need a code that have problems, exceptions, core panics :D

## Partition config

The last version of PlatformIO already include the partition file:

```bash
~/.platformio/packages/framework-arduinoespressif32/tools/partitions/default_16MB.csv
```

and this file already contains the **coredump** partition:

```bash
# Name,   Type, SubType, Offset,  Size, Flags
nvs,      data, nvs,     0x9000,  0x5000,
otadata,  data, ota,     0xe000,  0x2000,
app0,     app,  ota_0,   0x10000, 0x640000,
app1,     app,  ota_1,   0x650000,0x640000,
spiffs,   data, spiffs,  0xc90000,0x360000,
coredump, data, coredump,0xFF0000,0x10000,
```

that means that you only need in your manifest file this option, like this:

```cpp
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
board_build.partitions = default_16MB.csv ; <===== this ======
monitor_filters = 
  esp32_exception_decoder ; recommended
build_flags = 
  -D CORE_DEBUG_LEVEL=5   ; recommended only for debugging
```

Note: if your project use a different partition schema, you should change it to have this partition coredump, for instance adding one line more like this:

```bash
coredump, data, coredump,,        64K
```

## Code

In your setup or main funciton, you should have the validation and coredump initialization like this:

```cpp
#include "esp_core_dump.h"

void inline checkCoreDumpPartition() {
  esp_core_dump_init();
  esp_core_dump_summary_t *summary =
      static_cast<esp_core_dump_summary_t *>(malloc(sizeof(esp_core_dump_summary_t)));
  if (summary) {
    esp_err_t err = esp_core_dump_get_summary(summary);
    if (err == ESP_OK) {
      log_i("Getting core dump summary ok.");

    } else {
      log_e("Getting core dump summary not ok. Error: %d", (int)err);
      log_e("Probably no coredump present yet.");
      log_e("esp_core_dump_image_check() = %d", esp_core_dump_image_check());
    }
    free(summary);
  }
}

void setup(){
  checkCoreDumpPartition();
  // Your code..//
}
```

## esp-coredump installation

This utility is included in the tools of the IDF toolchain, but you may not have this tool if your project only uses the Arduino Framework and PlatformIO. Here, I found an alternative way to install this tool using the current PlatformIO setup, please follow the next steps:

### 1. Change to PlatformIO IDF installation directory**:

```bash
cd ~/.platformio/packages/framework-espidf
```

### 2. Perform the installation of Espressif toolchain

From this directory execute the next commads:

```bash
chmod 755 install.sh export.sh
./install.sh
```

this should install the tools in your home, specifically in `~/.espressif/`

### 3. Run `export.sh` like this:

```bash
. ./export.sh
```

Optional:

You can add the next alias configuration to your `.bashrc` or shell configuration, for future executions:

```bash
alias get_idf=". ~/.platformio/packages/framework-espidf/export.sh"
```

with that you should have the command `get_idf` that it will load the idf tools commands like esp-coredump.

### 4. Validate that the command was loaded:

```bash
esp-coredump --version
espcoredump.py v1.12.0
```

# Execution and analisys

In your project, after compiling and uploading your buggy code, if you encounter an exception, core panic, or any crash in your firmware, the coredump information will be in this partition. To retrieve the information, you should execute this tool in the root of your PlatformIO project, like this:

```bash
esp-coredump -p /dev/ttyACM0 info_corefile .pio/build/esp32dev/firmware.elf
```

In this command, -p specifies the USB port to which your device is connected, and the build directory corresponds to your env:xxxx declaration in your PlatformIO ini file.

Finishing, the output would contains the complete information of the exception and the core dump information, like this:

### Output:

<details>
<summary><strong>==> Click to expand <==</strong></summary>

```cpp
espcoredump.py v1.12.0
INFO: Invoke esptool to read image.
INFO: Retrieving core dump partition offset and size...
INFO: Core dump partition offset=4194304, size=65536
WARNING: The core dump image offset is not specified. Use partition offset: 0x400000.
INFO: esptool.py v4.8.1
Serial port /dev/ttyACM0
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (QFN56) (revision v0.2)
Features: WiFi, BLE, Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
MAC: 24:58:7c:e2:8b:b8
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
16 (100 %)
16 (100 %)
Read 16 bytes at 0x00400000 in 0.0 seconds (30.7 kbit/s)...
Hard resetting via RTS pin...

INFO: esptool.py v4.8.1
Serial port /dev/ttyACM0
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (QFN56) (revision v0.2)
Features: WiFi, BLE, Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
MAC: 24:58:7c:e2:8b:b8
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
12068 (100 %)
12068 (100 %)
Read 12068 bytes at 0x00400000 in 1.1 seconds (91.8 kbit/s)...
Hard resetting via RTS pin...

===============================================================
==================== ESP32 CORE DUMP START ====================

Crashed task handle: 0x3fcec088, name: 'loopTask', GDB name: 'process 1070514312'

================== CURRENT THREAD REGISTERS ===================
exccause       0x1c (LoadProhibitedCause)
excvaddr       0x4c
epc1           0x4200a7c5
epc2           0x0
epc3           0x0
epc4           0x0
epc5           0x0
epc6           0x0
eps2           0x0
eps3           0x0
eps4           0x0
eps5           0x0
eps6           0x0
pc             0x4201c5bc          0x4201c5bc <esp_now_is_peer_exist+20>
lbeg           0x40056f5c          1074098012
lend           0x40056f72          1074098034
lcount         0xffffffff          4294967295
sar            0x10                16
ps             0x60a20             395808
threadptr      <unavailable>
br             <unavailable>
scompare1      <unavailable>
acclo          <unavailable>
acchi          <unavailable>
m0             <unavailable>
m1             <unavailable>
m2             <unavailable>
m3             <unavailable>
expstate       <unavailable>
f64r_lo        <unavailable>
f64r_hi        <unavailable>
f64s           <unavailable>
fcr            <unavailable>
fsr            <unavailable>
a0             0x82002855          -2113918891
a1             0x3fcebe20          1070513696
a2             0x0                 0
a3             0x3fcef940          1070528832
a4             0x3fc9def0          1070194416
a5             0x3fc9def0          1070194416
a6             0x3fc95df0          1070161392
a7             0x0                 0
a8             0x0                 0
a9             0x3fcebdb0          1070513584
a10            0x1                 1
a11            0x3fcebdd0          1070513616
a12            0x4202fb3c          1107491644
a13            0x3fcebe48          1070513736
a14            0x40                64
a15            0x3                 3

==================== CURRENT THREAD STACK =====================
#0  0x4201c5bc in esp_now_is_peer_exist ()
#1  0x42002855 in sendMessage (msglen=249, mac=0x3fc95df0 <radio> "\377\377\377\377\377\377", <incomplete sequence \364>) at examples/lib/espcamlib/src/ESPNowCam.cpp:76
#2  0x42002a15 in ESPNowCam::sendData (this=0x3fc95df0 <radio>, data=<optimized out>, lenght=<optimized out>) at examples/lib/espcamlib/src/ESPNowCam.cpp:50
#3  0x42002387 in processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:31
#4  processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:26
#5  0x420024be in loop () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:90
#6  0x42004120 in loopTask (pvParameters=<optimized out>) at /home/avp/.platformio/packages/framework-arduinoespressif32/cores/esp32/main.cpp:50

======================== THREADS INFO =========================
  Id   Target Id          Frame 
* 1    process 1070514312 0x4201c5bc in esp_now_is_peer_exist ()
  2    process 1070550316 0x4202f5ea in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
  3    process 1070548916 0x4202f5ea in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
  4    process 1070541784 0x400559e0 in ?? ()
  5    process 1070536800 0x4037ed62 in vPortEnterCritical (mux=0x3fcf1438) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
  6    process 1070535100 0x4037ed64 in vPortEnterCritical (mux=0x3fcf0d94) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
  7    process 1070518840 0x4037ec02 in vPortEnterCritical (mux=0x3fcec904) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578

       TCB             NAME PRIO C/B  STACK USED/FREE
---------- ---------------- -------- ----------------
0x3fcec088         loopTask      1/1        2112/6072
0x3fcf4d2c            IDLE1      0/0          608/404
0x3fcf47b4            IDLE0      0/0          608/412
0x3fcf2bd8        esp_timer    22/22         640/3960
0x3fcf1860             ipc1    24/24          608/400
0x3fcf11bc             ipc0    24/24          608/404
0x3fced238         cam_task    23/23         656/1384

==================== THREAD 1 (TCB: 0x3fcec088, name: 'loopTask') =====================
#0  0x4201c5bc in esp_now_is_peer_exist ()
#1  0x42002855 in sendMessage (msglen=249, mac=0x3fc95df0 <radio> "\377\377\377\377\377\377", <incomplete sequence \364>) at examples/lib/espcamlib/src/ESPNowCam.cpp:76
#2  0x42002a15 in ESPNowCam::sendData (this=0x3fc95df0 <radio>, data=<optimized out>, lenght=<optimized out>) at examples/lib/espcamlib/src/ESPNowCam.cpp:50
#3  0x42002387 in processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:31
#4  processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:26
#5  0x420024be in loop () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:90
#6  0x42004120 in loopTask (pvParameters=<optimized out>) at /home/avp/.platformio/packages/framework-arduinoespressif32/cores/esp32/main.cpp:50

==================== THREAD 2 (TCB: 0x3fcf4d2c, name: 'IDLE1') =====================
#0  0x4202f5ea in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
#1  0x4200b449 in esp_vApplicationIdleHook () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/freertos_hooks.c:63
#2  0x4037f22b in prvIdleTask (pvParameters=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:4099

==================== THREAD 3 (TCB: 0x3fcf47b4, name: 'IDLE0') =====================
#0  0x4202f5ea in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
#1  0x4200b449 in esp_vApplicationIdleHook () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/freertos_hooks.c:63
#2  0x4037f22b in prvIdleTask (pvParameters=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:4099

==================== THREAD 4 (TCB: 0x3fcf2bd8, name: 'esp_timer') =====================
#0  0x400559e0 in ?? ()
#1  0x40380c75 in vPortClearInterruptMaskFromISR (prev_level=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:571
#2  vPortExitCritical (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/port.c:332
#3  0x40380511 in ulTaskGenericNotifyTake (uxIndexToWait=<optimized out>, xClearCountOnExit=1, xTicksToWait=4294967295) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:5513
#4  0x4200f9fd in timer_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:420

==================== THREAD 5 (TCB: 0x3fcf1860, name: 'ipc1') =====================
#0  0x4037ed62 in vPortEnterCritical (mux=0x3fcf1438) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueSemaphoreTake (xQueue=0x3fcf13ec, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1563
#2  0x40379eff in ipc_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_ipc/src/esp_ipc.c:54

==================== THREAD 6 (TCB: 0x3fcf11bc, name: 'ipc0') =====================
#0  0x4037ed64 in vPortEnterCritical (mux=0x3fcf0d94) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueSemaphoreTake (xQueue=0x3fcf0d48, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1563
#2  0x40379eff in ipc_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_ipc/src/esp_ipc.c:54

==================== THREAD 7 (TCB: 0x3fced238, name: 'cam_task') =====================
#0  0x4037ec02 in vPortEnterCritical (mux=0x3fcec904) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueReceive (xQueue=0x3fcec8b8, pvBuffer=0x3fced0b4, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1400
#2  0x42012424 in cam_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/components/esp32-camera/driver/cam_hal.c:125


======================= ALL MEMORY REGIONS ========================
Name   Address   Size   Attrs
.rtc.text 0x600fe000 0x21 R XA
.rtc.force_fast 0x600fe024 0x8 RW A
.rtc.force_slow 0x50000010 0x0 RW  
.iram0.vectors 0x40374000 0x403 R XA
.iram0.text 0x40374404 0x119cf R XA
.dram0.data 0x3fc95de0 0x4b68 RW A
.noinit 0x3fc9a948 0x0 RW  
.flash.text 0x42000020 0x30d2b R XA
.rtc.entry.literal 0x42030d4c 0x0 R XA
.flash.appdesc 0x3c040020 0x100 R  A
.flash.rodata 0x3c040120 0x11d3c RW A
.iram0.text_end 0x40385dd3 0x0 RW  
.iram0.bss 0x40385dd4 0x0 RW  
.dram0.heap_start 0x3fc9e7c8 0x0 RW  
.coredump.tasks.data 0x3fcec088 0x158 RW 
.coredump.tasks.data 0x3fcebd60 0x310 RW 
.coredump.tasks.data 0x3fcf4d2c 0x158 RW 
.coredump.tasks.data 0x3fcf4ab0 0x260 RW 
.coredump.tasks.data 0x3fcf47b4 0x158 RW 
.coredump.tasks.data 0x3fcf4540 0x260 RW 
.coredump.tasks.data 0x3fcf2bd8 0x158 RW 
.coredump.tasks.data 0x3fcf2940 0x280 RW 
.coredump.tasks.data 0x3fcf1860 0x158 RW 
.coredump.tasks.data 0x3fcf15e0 0x260 RW 
.coredump.tasks.data 0x3fcf11bc 0x158 RW 
.coredump.tasks.data 0x3fcf0f40 0x260 RW 
.coredump.tasks.data 0x3fced238 0x158 RW 
.coredump.tasks.data 0x3fcecf90 0x290 RW 

===================== ESP32 CORE DUMP END =====================
===============================================================
Done!
```
</details>

(You also can download this coredump output example from [here](https://termbin.com/ddur) to better view)

### Analisys

To better understand this information, you can watch the video [^2] included in the #references section. However, to summarize, you can focus on this section:

```cpp
========= THREAD 1 (TCB: 0x3fcec088, name: 'loopTask') ==========
#0  0x4201c5bc in esp_now_is_peer_exist ()
#1  0x42002855 in sendMessage (msglen=249, mac=0x3fc95df0 <radio> "\377\377\377\377\377\377", <incomplete sequence \364>) at examples/lib/espcamlib/src/ESPNowCam.cpp:76
#2  0x42002a15 in ESPNowCam::sendData (this=0x3fc95df0 <radio>, data=<optimized out>, lenght=<optimized out>) at examples/lib/espcamlib/src/ESPNowCam.cpp:50
#3  0x42002387 in processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:31
#4  processFrame () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:26
#5  0x420024be in loop () at examples/xiao-fpv-sender/xiao-fpv-sender.cpp:90
```

It shows the lines where the main thread, specifically the loop thread of the Arduino code, is crashing. For instance, it indicates that the **xiao-fpv-sender** example calls other functions from a library that caused the problem; in this case, the library object was never initialized (I forced this by commenting out the initialization).

# Troubleshooting

If you don't have this directory:

```bash
cd ~/.platformio/packages/framework-espidf
```

You only run a basic hello world in PlatformIO using `framework = espidf`.

# References

[^1]: [Espressif Coredump documentation](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/core_dump.html)  
[^2]: [Video: Using the core dump via Serial with IDF](https://youtu.be/MpD_3oVJAEs?si=xyctwDvsdBflVIRM)
