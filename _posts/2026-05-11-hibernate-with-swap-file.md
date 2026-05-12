---
layout: post
title:  "Hibernate using swap file"
date:   2026-03-21
excerpt: "Fast how to the how enable hibernate using swap files instead disk partition"
tag:
- GNU-Linux
- Debian
comments: false
---

## When you need that?

When maybe you chosen a small partition for swap and you need enable the hibernation feature that it needs the same amount of your RAM memory, but you don't have it and you don't want do changes in your disk partitions.

## Swap file creation

```bash
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

This could be works for instance for a system with 16Gb of RAM.

## Fstab config

In your `/etc/fstab` add the target path of your new swap file in fstab, and save the UUID of the partition that store this file, for instance, in my system the swap file is saved in my root partition or /, and this partition has this UUID=xxxxx-yyyy. Save this for the next steps.

```bash
# / during installation
UUID=xxxxx-yyyyy /                ext4    errors=remount-ro     0       1
# old swap during installation
UUID=zzzzz-pppppp none            swap    defaults              0       0
# new swap file
/swapfile         none            swap    defaults              0       0
```

## Swap resume offset

With the base partition of this `swapfile`, we need also the offset of the allocation of this file. For that we run the next `filefrag` tool and take the first big number, like this:

```bash
sudo filefrag -v /swapfile
Filesystem type is: ef53
File size of /swapfile is 17179869184 (4194304 blocks of 4096 bytes)
 ext:     logical_offset:        physical_offset: length:   expected: flags:
   0:        0..       0:   14120960..  14120960:      1:            
   1:        1..    2047:   14120961..  14123007:   2047:             unwritten
   2:     2048..    4095:   24791040..  24793087:   2048:   14123008: unwritten
...
```

in this case the number is **14120960**, and this the `resume_offset` for the next step:

## Grub config

On `/etc/default/grub` please add the next line:

```conf
GRUB_CMDLINE_LINUX_DEFAULT="quiet resume=UUID=xxxx-yyyy resume_offset=14120960"
```

## Allow Hibernation

Please check in `/etc/systemd/sleep.conf` something like that:

```conf
[Sleep]
AllowSuspend=yes
AllowHibernation=yes
AllowSuspendThenHibernate=yes
HibernateState=disk
```

## Testing

Use the command `pm-hibernate` or `systemctl hibernate`

## Troubleshooting

### Hibernate not supported

```bash
sudo systemctl hibernate
Call to Hibernate failed: Sleep verb "hibernate" not supported
```

In this case is possible that you have enable secure boot in your bios, maybe you need disable this.

### Hibernate start but it exit

```bash
[ 2103.573204] PM: Cannot find swap device, try swapon -a
[ 2103.573216] PM: Cannot get swap writer
[ 2103.706155] PM: hibernation: Basic memory bitmaps freed
[ 2103.706682] OOM killer enabled.
[ 2103.706683] Restarting tasks ... done.
[ 2103.728915] PM: hibernation: hibernation exit
```

In this case you should double check the resume_offset step. Remember that `resume=UUID=xxxx` should be the UUID of the partition of your swap file.
