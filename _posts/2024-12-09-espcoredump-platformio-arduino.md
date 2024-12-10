---
layout: post
title:  "ESPCoredump with Arduino Framework using PlatformIO"
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

The latest version of PlatformIO and Espressif for the Arduino Framework includes the option for Coredump in FLASH. This means that you can capture exceptions with more information even when you are not connected to the device. The coredump binaries will be stored in your flash memory, allowing you to retrieve and analyze this information later. Additionally, this coredump provides more information than the standard exception decoder filter in PlatformIO, such as the complete stack backtrace, the current tasks at the moment of the crash, and other relevant details. Previously, this option was only available for IDF projects, but it is now possible with the Arduino Framework. This guide aims to explain what you need to get started.

## Prerequisites

I assume that you are familiar with PlatformIO and have already deployed any firmware or app using the Arduino framework option.

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

Note: if your project use a different partition schema, you should change it to have this partition coredump.

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

This utility is included in the tools of the IDF toolchain, but you may not have this tool if your project only uses the Arduino Framework and PlatformIO. Here, I found an alternative way to install this tool using the current PlatformIO setup:

1. Change to PlatformIO IDF installation directory**:

```bash
cd ~/.platformio/packages/framework-espidf
```

2. Perform the installation of Espressif toolchain from there following the next steps:

```bash
chmod 755 install.sh export.sh
./install.sh
```

this should install the tools in your home, in the directory `~/.espressif/`

3. Run `export.sh` like this:

```bash
. ./export.sh
```

Optional:

You can add this alias configuration to your `.bashrc` or shell configuration file like this:

```bash
alias get_idf=". ~/.platformio/packages/framework-espidf/export.sh"
```

4. Validate that the command was installed by running: 

```bash
esp-coredump --version
espcoredump.py v1.12.0
```

## Execution and analisys

In your project, after compiling and uploading your buggy code, if you encounter an exception, core panic, or any crash in your firmware, the coredump information will be in this partition. To retrieve the information, you should execute this tool in the root of your PlatformIO project like this:

```bash
esp-coredump -p /dev/ttyACM0 info_corefile .pio/build/esp32dev/firmware.elf
```

In this command, -p specifies the USB port to which your device is connected, and the build directory corresponds to your env:xxxx declaration in your PlatformIO ini file.

And the output could contains the complete information of the exception and the coredump information like this:

```cpp
espcoredump.py v1.12.0
INFO: Invoke esptool to read image.
INFO: Retrieving core dump partition offset and size...
INFO: Core dump partition offset=16711680, size=65536
WARNING: The core dump image offset is not specified. Use partition offset: 0xff0000.
INFO: esptool.py v4.8.1
Serial port /dev/ttyACM0
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (QFN56) (revision v0.1)
Features: WiFi, BLE, Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
MAC: 34:85:18:99:b8:80
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
16 (100 %)
16 (100 %)
Read 16 bytes at 0x00ff0000 in 0.0 seconds (30.9 kbit/s)...
Hard resetting via RTS pin...

INFO: esptool.py v4.8.1
Serial port /dev/ttyACM0
Connecting...
Detecting chip type... ESP32-S3
Chip is ESP32-S3 (QFN56) (revision v0.1)
Features: WiFi, BLE, Embedded PSRAM 8MB (AP_3v3)
Crystal is 40MHz
MAC: 34:85:18:99:b8:80
Uploading stub...
Running stub...
Stub running...
Configuring flash size...
17284 (100 %)
17284 (100 %)
Read 17284 bytes at 0x00ff0000 in 1.5 seconds (91.7 kbit/s)...
Hard resetting via RTS pin...

===============================================================
==================== ESP32 CORE DUMP START ====================

Crashed task handle: 0x3fcf2fa8, name: 'esp_timer', GDB name: 'process 1070542760'

================== CURRENT THREAD REGISTERS ===================
exccause       0x1d (StoreProhibitedCause)
excvaddr       0x0
epc1           0x4206e9dd
epc2           0x0
epc3           0x0
epc4           0x4217bf36
epc5           0x0
epc6           0x0
eps2           0x0
eps3           0x0
eps4           0x60220
eps5           0x0
eps6           0x0
pc             0x40378475          0x40378475 <panic_abort+21>
lbeg           0x40056f5c          1074098012
lend           0x40056f72          1074098034
lcount         0x0                 0
sar            0x4                 4
ps             0x60721             395041
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
a0             0x8037ed18          -2143818472
a1             0x3fc99780          1070176128
a2             0x3fc997ea          1070176234
a3             0x3fc99817          1070176279
a4             0xa                 10
a5             0x31                49
a6             0x0                 0
a7             0x3fc99725          1070176037
a8             0x0                 0
a9             0x1                 1
a10            0x3fc997ce          1070176206
a11            0x3fc997ce          1070176206
a12            0xa                 10
a13            0x0                 0
a14            0x2c9a0c0           46768320
a15            0xffffff            16777215

==================== CURRENT THREAD STACK =====================
#0  0x40378475 in panic_abort (details=0x3fc997ea "abort() was called at PC 0x4206f904 on core 0") at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/panic.c:408
#1  0x4037ed18 in esp_system_abort (details=0x3fc997ea "abort() was called at PC 0x4206f904 on core 0") at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/esp_system.c:137
#2  0x40385dc0 in abort () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/newlib/abort.c:46
#3  0x4206f907 in task_wdt_isr (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/task_wdt.c:176
#4  0x4037a5bc in _xt_lowint1 () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/xtensa_vectors.S:1118
#5  0x400559e0 in ?? ()
#6  0x40382281 in vPortClearInterruptMaskFromISR (prev_level=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:571
#7  vPortExitCritical (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/port.c:332
#8  0x40379e11 in vPortExitCriticalSafe (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:607
#9  timer_list_unlock (timer_type=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:346
#10 0x42092300 in timer_process_alarm (dispatch_method=ESP_TIMER_TASK) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:395
#11 timer_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:422

======================== THREADS INFO =========================
  Id   Target Id          Frame 
* 1    process 1070542760 0x40378475 in panic_abort (details=0x3fc997ea "abort() was called at PC 0x4206f904 on core 0") at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/panic.c:408
  2    process 1070525548 0x4204e024 in gpsTask (pvParameters=<optimized out>) at lib/tasks/tasks.cpp:44
  3    process 1070549956 0x4217bf36 in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
  4    process 1070551356 0x4217bf36 in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
  5    process 1070514456 0x4200c7ca in loop () at src/main.cpp:202
  6    process 1070560056 cliTask (param=<optimized out>) at lib/tasks/tasks.cpp:69
  7    process 1070561128 0x4037fdea in vPortEnterCritical (mux=0x3fcf74ec) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
  8    process 1070535160 0x4037ff4c in vPortEnterCritical (mux=0x3fcf0dd0) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
  9    process 1070536860 0x4037ff4c in vPortEnterCritical (mux=0x3fcf1474) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
  10   process 1070524240 0x400559e0 in ?? ()


       TCB             NAME PRIO C/B  STACK USED/FREE
---------- ---------------- -------- ----------------
0x3fcf2fa8        esp_timer    22/22         640/3960
0x3fceec6c         GPS Task      1/1         656/7520
0x3fcf4bc4            IDLE0      0/0          608/412
0x3fcf513c            IDLE1      0/0          608/404
0x3fcec118         loopTask      1/1         576/7608
0x3fcf7338         cliTask       1/1        560/19424
0x3fcf7768          sys_evt    20/20         688/1860
0x3fcf11f8             ipc0    24/24          608/408
0x3fcf189c             ipc1    24/24          608/404
0x3fcee750         lvglDraw      3/3         688/7496

==================== THREAD 1 (TCB: 0x3fcf2fa8, name: 'esp_timer') =====================
#0  0x40378475 in panic_abort (details=0x3fc997ea "abort() was called at PC 0x4206f904 on core 0") at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/panic.c:408
#1  0x4037ed18 in esp_system_abort (details=0x3fc997ea "abort() was called at PC 0x4206f904 on core 0") at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/esp_system.c:137
#2  0x40385dc0 in abort () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/newlib/abort.c:46
#3  0x4206f907 in task_wdt_isr (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/task_wdt.c:176
#4  0x4037a5bc in _xt_lowint1 () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/xtensa_vectors.S:1118
#5  0x400559e0 in ?? ()
#6  0x40382281 in vPortClearInterruptMaskFromISR (prev_level=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:571
#7  vPortExitCritical (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/port.c:332
#8  0x40379e11 in vPortExitCriticalSafe (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:607
#9  timer_list_unlock (timer_type=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:346
#10 0x42092300 in timer_process_alarm (dispatch_method=ESP_TIMER_TASK) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:395
#11 timer_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_timer/src/esp_timer.c:422

==================== THREAD 2 (TCB: 0x3fceec6c, name: 'GPS Task') =====================
#0  0x4204e024 in gpsTask (pvParameters=<optimized out>) at lib/tasks/tasks.cpp:44

==================== THREAD 3 (TCB: 0x3fcf4bc4, name: 'IDLE0') =====================
#0  0x4217bf36 in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
#1  0x4206f694 in esp_vApplicationIdleHook () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/freertos_hooks.c:63
#2  0x40380413 in prvIdleTask (pvParameters=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:4099

==================== THREAD 4 (TCB: 0x3fcf513c, name: 'IDLE1') =====================
#0  0x4217bf36 in esp_pm_impl_waiti () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_pm/pm_impl.c:855
#1  0x4206f694 in esp_vApplicationIdleHook () at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_system/freertos_hooks.c:63
#2  0x40380413 in prvIdleTask (pvParameters=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:4099

==================== THREAD 5 (TCB: 0x3fcec118, name: 'loopTask') =====================
#0  0x4200c7ca in loop () at src/main.cpp:202
#1  0x4205fd18 in loopTask (pvParameters=<optimized out>) at /home/avp/.platformio/packages/framework-arduinoespressif32/cores/esp32/main.cpp:50

==================== THREAD 6 (TCB: 0x3fcf7338, name: 'cliTask ') =====================
#0  cliTask (param=<optimized out>) at lib/tasks/tasks.cpp:69

==================== THREAD 7 (TCB: 0x3fcf7768, name: 'sys_evt') =====================
#0  0x4037fdea in vPortEnterCritical (mux=0x3fcf74ec) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueReceive (xQueue=0x3fcf74a0, pvBuffer=0x3fcf3c90, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1400
#2  0x42183b85 in esp_event_loop_run (event_loop=0x3fceedd4, ticks_to_run=4294967295) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_event/esp_event.c:566
#3  0x42183d03 in esp_event_loop_run_task (args=0x3fceedd4) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_event/esp_event.c:115

==================== THREAD 8 (TCB: 0x3fcf11f8, name: 'ipc0') =====================
#0  0x4037ff4c in vPortEnterCritical (mux=0x3fcf0dd0) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueSemaphoreTake (xQueue=0x3fcf0d84, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1563
#2  0x4037abdb in ipc_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_ipc/src/esp_ipc.c:54

==================== THREAD 9 (TCB: 0x3fcf189c, name: 'ipc1') =====================
#0  0x4037ff4c in vPortEnterCritical (mux=0x3fcf1474) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:578
#1  xQueueSemaphoreTake (xQueue=0x3fcf1428, xTicksToWait=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/queue.c:1563
#2  0x4037abdb in ipc_task (arg=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/esp_ipc/src/esp_ipc.c:54

==================== THREAD 10 (TCB: 0x3fcee750, name: 'lvglDraw') =====================
#0  0x400559e0 in ?? ()
#1  0x40382281 in vPortClearInterruptMaskFromISR (prev_level=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/include/freertos/portmacro.h:571
#2  vPortExitCritical (mux=<optimized out>) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/port/xtensa/port.c:332
#3  0x403819d9 in ulTaskGenericNotifyTake (uxIndexToWait=<optimized out>, xClearCountOnExit=1, xTicksToWait=4294967295) at /home/runner/work/esp32-arduino-lib-builder/esp32-arduino-lib-builder/esp-idf/components/freertos/tasks.c:5513
#4  0x420353da in lv_thread_sync_wait (pxCond=0x3d808218) at .pio/libdeps/TDECK_ESP32S3/lvgl/src/osal/lv_freertos.c:215
#5  0x4202d645 in render_thread_cb (ptr=0x3d8081f8) at .pio/libdeps/TDECK_ESP32S3/lvgl/src/draw/sw/lv_draw_sw.c:482
#6  0x4203529a in prvRunThread (pxArg=0x3d808224) at .pio/libdeps/TDECK_ESP32S3/lvgl/src/osal/lv_freertos.c:431


======================= ALL MEMORY REGIONS ========================
Name   Address   Size   Attrs
.rtc.text 0x600fe000 0x41 R XA
.rtc.force_fast 0x600fe044 0x8 RW A
.rtc.force_slow 0x50000010 0x0 RW  
.iram0.vectors 0x40374000 0x403 R XA
.iram0.text 0x40374404 0x13b0b R XA
.dram0.data 0x3fc97f10 0x641c RW A
.noinit 0x3fc9e32c 0x0 RW  
.flash.text 0x42000020 0x18531b R XA
.rtc.entry.literal 0x4218533c 0x0 R XA
.flash.appdesc 0x3c190020 0x100 R  A
.flash.rodata 0x3c190120 0xbbee4 RW A
.iram0.text_end 0x40387f0f 0x0 RW  
.iram0.bss 0x40387f10 0x0 RW  
.dram0.heap_start 0x3fcb36d0 0x0 RW  
.coredump.tasks.data 0x3fcf2fa8 0x158 RW 
.coredump.tasks.data 0x3fcf2d10 0x280 RW 
.coredump.tasks.data 0x3fceec6c 0x158 RW 
.coredump.tasks.data 0x3fcb58c0 0x290 RW 
.coredump.tasks.data 0x3fcf4bc4 0x158 RW 
.coredump.tasks.data 0x3fcf4950 0x260 RW 
.coredump.tasks.data 0x3fcf513c 0x158 RW 
.coredump.tasks.data 0x3fcf4ec0 0x260 RW 
.coredump.tasks.data 0x3fcec118 0x158 RW 
.coredump.tasks.data 0x3fcebec0 0x240 RW 
.coredump.tasks.data 0x3fcf7338 0x158 RW 
.coredump.tasks.data 0x3fcba750 0x230 RW 
.coredump.tasks.data 0x3fcf7768 0x158 RW 
.coredump.tasks.data 0x3fcf3b80 0x2b0 RW 
.coredump.tasks.data 0x3fcf11f8 0x158 RW 
.coredump.tasks.data 0x3fcf0f80 0x260 RW 
.coredump.tasks.data 0x3fcf189c 0x158 RW 
.coredump.tasks.data 0x3fcf1620 0x260 RW 
.coredump.tasks.data 0x3fcee750 0x158 RW 
.coredump.tasks.data 0x3fcf7040 0x2b0 RW 

===================== ESP32 CORE DUMP END =====================
===============================================================
Done!
```

# Troubleshooting

** If you don't have the directory:

```bash
cd ~/.platformio/packages/framework-espidf
```

then, you only run a basic hello world in PlatformIO using `framework = espidf`.

# Credits

[Arch Documentation](https://wiki.archlinux.org/title/Extreme_Multihead)  
[Android Tablet Script](https://gist.github.com/8bitbuddhist/7ab180286d6eb7fdc68d374063814175)

