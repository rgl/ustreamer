# µStreamer
[![CI](https://github.com/pikvm/ustreamer/workflows/CI/badge.svg)](https://github.com/pikvm/ustreamer/actions?query=workflow%3ACI)
[![Discord](https://img.shields.io/discord/580094191938437144?logo=discord)](https://discord.gg/bpmXfz5)

[[Русская версия]](README.ru.md)

µStreamer is a lightweight and very quick server to stream [MJPG](https://en.wikipedia.org/wiki/Motion_JPEG) video from any V4L2 device to the net. All new browsers have native support of this video format, as well as most video players such as mplayer, VLC etc.
µStreamer is a part of the [Pi-KVM](https://github.com/pikvm/pikvm) project designed to stream [VGA](https://www.amazon.com/dp/B0126O0RDC) and [HDMI](https://auvidea.com/b101-hdmi-to-csi-2-bridge-15-pin-fpc/) screencast hardware data with the highest resolution and FPS possible.

µStreamer is very similar to [mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer) with ```input_uvc.so``` and ```output_http.so``` plugins, however, there are some major differences. The key ones are:

| **Feature** | **µStreamer** | **mjpg-streamer** |
|----------|---------------|-------------------|
| Multithreaded JPEG encoding | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| [OpenMAX IL](https://www.khronos.org/openmaxil) hardware acceleration<br>on Raspberry Pi | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| Behavior when the device<br>is disconnected while streaming | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Shows a black screen<br>with ```NO SIGNAL``` on it<br>until reconnected | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) Stops the streaming <sup>1</sup> |
| [DV-timings](https://linuxtv.org/downloads/v4l-dvb-apis-new/userspace-api/v4l/dv-timings.html) support -<br>the ability to change resolution<br>on the fly by source signal | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#ffaa00](https://placehold.it/15/ffaa00/000000?text=+) Partially yes <sup>1</sup> |
| Option to skip frames when streaming<br>static images by HTTP to save the traffic | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes <sup>2</sup> | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| Streaming via UNIX domain socket | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| Debug logs without recompiling,<br>performance statistics log,<br>access to HTTP streaming parameters | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| Option to serve files<br>with a built-in HTTP server | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#ffaa00](https://placehold.it/15/ffaa00/000000?text=+) Regular files only |
| Signaling about the stream state<br>on GPIO using [libgpiod](https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/about) | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No |
| Access to webcam controls (focus, servos)<br>and settings such as brightness via HTTP | ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) No | ![#00aa00](https://placehold.it/15/00aa00/000000?text=+) Yes |

Footnotes:
  * ```1``` Long before µStreamer, I made a [patch](https://github.com/jacksonliam/mjpg-streamer/pull/164) to add DV-timings support to mjpg-streamer and to keep it from hanging up no device disconnection. Alas, the patch is far from perfect and I can't guarantee it will work every time - mjpg-streamer's source code is very complicated and its structure is hard to understand. With this in mind, along with needing multithreading and JPEG hardware acceleration in the future, I decided to make my own stream server from scratch instead of supporting legacy code.
  
  * ```2``` This feature allows to cut down outgoing traffic several-fold when streaming HDMI, but it increases CPU usage a little bit. The idea is that HDMI is a fully digital interface and each captured frame can be identical to the previous one byte-wise. There's no need to stream the same image over the net several times a second. With the `--drop-same-frames=20` option enabled, µStreamer will drop all the matching frames (with a limit of 20 in a row). Each new frame is matched with the previous one first by length, then using ```memcmp()```.

-----
# TL;DR
If you're going to live-stream from your backyard webcam and need to control it, use mjpg-streamer. If you need a high-quality image with high FPS - µStreamer for the win.

-----
# Building
You'll need  ```make```, ```gcc```, ```libevent``` with ```pthreads``` support, ```libjpeg8```/```libjpeg-turbo```, ```libuuid``` and ```libbsd``` (only for Linux).

* Arch: `sudo pacman -S libevent libjpeg-turbo libutil-linux libbsd`.
* Raspbian: `sudo apt install libevent-dev libjpeg8-dev uuid-dev libbsd-dev`. Add `libraspberrypi-dev` for `WITH_OMX=1` and `libgpiod` for `WITH_GPIO=1`.
* Debian: `sudo apt install build-essential libevent-dev libjpeg62-turbo-dev uuid-dev libbsd-dev`.

On Raspberry Pi you can build the program with OpenMAX IL. To do this pass option ```WITH_OMX=1``` to ```make```. To enable GPIO support install [libgpiod](https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/about) and pass option ```WITH_GPIO=1```. If the compiler reports about a missing function ```pthread_get_name_np()``` (or similar), add option ```WITH_PTHREAD_NP=0``` (it's enabled by default). For the similar error with ```setproctitle()``` add option ```WITH_SETPROCTITLE=0```.

```
$ git clone --depth=1 https://github.com/pikvm/ustreamer
$ cd ustreamer
$ make
$ ./ustreamer --help
```

AUR has a package for Arch Linux: https://aur.archlinux.org/packages/ustreamer. It should compile automatically with OpenMAX IL on Raspberry Pi, if the corresponding headers are present in ```/opt/vc/include```.  
FreeBSD port: https://www.freshports.org/multimedia/ustreamer.

## Building (Docker on Raspbian on Raspberry Pi 4)

Build the base raspbian image:

```bash
git clone https://github.com/debuerreotype/debuerreotype.git
cd debuerreotype
./raspbian.sh output buster
docker import output/raspbian/armhf/buster/slim/rootfs.tar.xz raspbian:buster-slim
```

Build ustreamer:

```bash
docker build --file pkg/docker/Dockerfile --tag ustreamer .
```

Set the hdmi-to-csi bridge EDID:

```bash
wget -qO tc358743-edid.hex https://raw.githubusercontent.com/pikvm/kvmd/master/configs/kvmd/tc358743-edid.hex
v4l2-ctl --device /dev/video0 --set-edid file=tc358743-edid.hex --fix-edid-checksums
```

Start ustreamer in foreground:

```bash
docker run --rm \
  --device /dev/video0 \
  --device /dev/vchiq \
  -p 8080:8080 \
  ustreamer \
  --device=/dev/video0 \
  --persistent \
  --dv-timings \
  --format=uyvy \
  --encoder=omx \
  --workers=$(nproc) \
  --quality=85 \
  --desired-fps=30 \
  --drop-same-frames=30 \
  --slowdown \
  --host=0.0.0.0 \
  --port=8080
```

Then access the web interface at port 8080 (e.g. http://raspberrypi.local:8080).

-----
# Usage
Without arguments, ```ustreamer``` will try to open ```/dev/video0``` with 640x480 resolution and start streaming on  ```http://127.0.0.1:8080```. You can override this behavior using parameters ```--device```, ```--host``` and ```--port```. For example, to stream to the world, run:
```
# ./ustreamer --device=/dev/video1 --host=0.0.0.0 --port=80
```

:exclamation: Please note that since µStreamer v2.0 cross-domain requests were disabled by default for [security reasons](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS). To enable the old behavior, use the option `--allow-origin=\*`.

The recommended way of running µStreamer with [Auvidea B101](https://www.raspberrypi.org/forums/viewtopic.php?f=38&t=120702&start=400#p1339178) on Raspberry Pi:
```bash
$ ./ustreamer \
    --format=uyvy \ # Device input format
    --encoder=omx \ # Hardware encoding with OpenMAX
    --workers=3 \ # Maximum workers for OpenMAX
    --persistent \ # Don't re-initialize device on timeout (for example when HDMI cable was disconnected)
    --dv-timings \ # Use DV-timings
    --drop-same-frames=30 # Save the traffic
```

:exclamation: Please note that to use `--drop-same-frames` for different browsers you need to use some specific URL `/stream` parameters (see URL `/` for details).

You can always view the full list of options with ```ustreamer --help```.

-----
# See also
* [Running uStreamer via systemd service](https://github.com/pikvm/ustreamer/issues/16).
* [uStreamer Ansible Role](https://github.com/mtlynch/ansible-role-ustreamer): Use [Ansible](https://docs.ansible.com/ansible/latest/index.html) to compile uStreamer and install it as a systemd service automatically.

-----
# License
Copyright (C) 2018 by Maxim Devaev mdevaev@gmail.com

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see https://www.gnu.org/licenses/.
