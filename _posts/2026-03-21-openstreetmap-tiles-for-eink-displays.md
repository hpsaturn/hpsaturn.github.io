---
layout: post
title:  "OpenStreetMap tiles for eInk displays"
date:   2026-03-21
excerpt: "Alternative to pay APIs for get scale grays map tiles optimized for eInk displays"
feature: "https://hpsaturn.com/assets/img/picocalc_x48ng_preview.jpg"
tag:
- GNU-Linux
- OSM
- ESP32
comments: false
---

## Map tiles for eInk displays

On some map apps using microcontrollers and eInk displays, like Meshtastic or Meshcore, you need special map tiles, but some guides using restricted API to download maps with special rules for improve the tiles for this kind of screens, the problem is that in some cases these APIs have a free quote or don't have all zones.

This alternative use Maperitive without tricky rules, only the default rule and one Python script to post-procesing these tiles. That is also useful for previous downloaded tiles, for instance, your old color tiles for your TFT screens.

Also in this guide, I included some tips for Linux, and some improvements in the copy of these tiles to the SDCard of the device. All here was tested on a LilyGO T-Deck Pro that has eInk display.

## Maperitive installation (Linux)

Download first [Maperative](http://maperitive.net/). This is a software only for Windows but we have some alternatives here for running it on Linux:

### docker-wine

This version of wine is easy to launch and it contains everthing already tunned for have a good desktop integration. Download [docker-wine](https://hub.docker.com/r/scottyhardy/docker-wine/) and run it on the Maperitive directory like this:

```bash
cd Maperitive
docker-wine --as-root --force-owner --volume=$PWD:/data wine /data/Maperitive.exe
```

### mono

Other alternative to run Maperitive is using [Mono libraries](https://wiki.openstreetmap.org/wiki/Maperitive#Ubuntu_10.04_-_12.10). But the disadvantage of this option is that you should install these libraries in your system. With docker-wine all stuff will be in an image that you could remove in any moment without affect your system. After install mono run this:

```bash
cd Maperitive
chmod +x ./Maperitive.sh
./Maperitive.sh
```

## Tiles Generation

After launch Maperitive your able to select your zone and generate your tiles. For that enter to `MAP-> Set Geometry bounds` and then draw or expand the square of your zone and run the next commands in the `Command prompt` box:

(only for docker-wine option:)

```bash
change-dir /data/
```

tiles generation:

```bash
generate-tiles minzoom=6 maxzoom=17
```

it could takes long time, maybe 1 hour or more depending your area and zoom. After that you should have your Tiles in the Maperitive directory.

{% capture images %}
  {{ site.url }}/assets/img/maperitive_tiles_generation.jpg
{% endcapture %}
{% include gallery images=images cols=1 %}

## Grey Scale Tiles Convertion

Now we could convert these tiles to an optimized version for eInk display, for instance for our T-Deck Pro. For that download this script [osm_tile_to_eink.py](https://gist.github.com/hpsaturn/757f4ffb77f97570b42d355e355120e9) and run it like this:

```bash
python osm_tile_to_eink.py Tiles tiles_grey --mode bw
```

the `Tiles` directory should be in Maperitive directory. And then will be not modified. You are able to backup these for instance for TFT displays or other color devices.

## Transfer Tiles to SD

Depending what firmware or device you are using, you should copy this tiles in the root of its SD card, for instance for Meshcore, it should be in `tiles` directory in the root of the SD.

The copy could be so slow because are many files. For that, here I shared some alternatives:

### rsync

It is better than the normal `cp` command, but also could be taking many time in the process.

```bash
rsync -av --progress /path/to/source/Tiles /mnt/sd_image/tiles
```

### SD card image (recommended)

To improve to the maxium speed you could make first an full image with all tiles and copy there all tiles, and then burn this image in the SD card partion. For that follow the next commands:

create a empty image:

```bash
truncate -s 16G sd_image.img
```

format the empty image. For instance for Meshcore:

```bash
sudo mkfs.vfat -F 32 -n "MeshCore2" sd_image.img
```

mount the image:

```bash
mkdir /mnt/img
sudo mount -o loop,uid=$(id -u),gid=$(id -g),umask=000 /tmp/sd_image.img /mnt/img
```

copy the tiles. For instance for Meshcore:

```bash
rsync -av --progress tiles_grey /mnt/img
mv /mnt/img/tiles_grey /mnt/img/tiles
sudo umount /mnt/img
```

## Results

{% capture images %}
  {{ site.url }}/assets/img/meshcore_map_tiles_preview00.jpg
  {{ site.url }}/assets/img/meshcore_map_tiles_preview01.jpg
  {{ site.url }}/assets/img/meshcore_map_tiles_preview02.jpg
{% endcapture %}
{% include gallery images=images cols=3 %}
