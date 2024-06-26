---
layout: post
title:  "MatrixCreator PocketSphinx"
date:   2016-10-18
excerpt: "PocketSphinx migration to MATRIXCreator board."
feature: http://hpsaturn.com/assets/img/matrixcreatorheader.png
tag:
- MatrixVoice
- MatrixCreator
- PocketSphinx
- C++
- IoT
- Voice Assistant
- Protobuf
- ZeroMQ
- Wakeword
comments: false
---

# MatrixCreator PocketSphinx

Continuous voice detection using `PocketSphinx` and `MATRIXCreator` board.  

## Compile from sources

Please review the last version of this document in [github](https://github.com/matrix-io/matrix-creator-pocketsphinx).

### Step 1: MatrixCreator Software

``` bash 
curl https://apt.matrix.one/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.matrix.one/raspbian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/matrixlabs.list
sudo apt update
sudo apt upgrade

sudo apt install matrixio-creator-init matrixio-kernel-modules libmatrixio-creator-hal-dev matrixio-pocketsphinx
sudo reboot
```

### Step 2: Building PocketSphinx demos

```bash
git clone https://github.com/matrix-io/matrix-creator-pocketsphinx.git
cd matrix-creator-pocketsphinx
mkdir build && cd build && cmake .. && make -j $(nproc)
```

### Step 3: Install testing voice commands:

Download sample language and dictionary from [here](https://drive.google.com/file/d/0B3lA7p7SjZu-YUJxYmIwcnh4Qlk/view?usp=sharing) and transfer it to your Pi on `matrix-creator-pocketsphinx/build/demos` directory and then extract it:

```bash
mkdir assets
tar zxf TAR6706.tgz -C assets
```

**NOTE**: Optional, you can make new models [explanation below](https://github.com/matrix-io/matrix-creator-pocketsphinx#optional-custom-lenguage-and-phrases-for-recognition)

### Step 4: Run DEMO:

on build/demos:

```bash
./pocketsphinx_demo -keyphrase "MATRIX" -kws_threshold 1e-20 -dict assets/6706.dic -lm assets/6706.lm -inmic yes
```

and try it with executing commands with your voice like this: 

- `matrix everloop`
- `matrix stop`
- `matrix clear`
- ...

### (optional) Custom lenguage and phrases for voice recognition 

1. Make a text plane like this:

```bash
matrix
everloop
arc 
clear
stop
shutdown
now
ipaddress
matrix everloop
matrix clear
matrix stop
matrix ipaddress
matrix game time
matrix one minute
matrix two minutes
matrix three minutes
matrix four minutes
matrix five minutes
matrix ten seconds
matrix ten minutes
```

2. Upload this file to [Sphinx Knowledge Base Tool](http://www.speech.cs.cmu.edu/tools/lmtool-new.html) and compile knowledge base.

3. Dowload *TARXXXXX.tgz* and upgrade assets.
