
# GStreamer pipeline examples with thead OpenMax-IL lib

Some GStreamer pipeline command-line examples with **omxil library** on **TH1520**, **RevyOS**

## Get ready

In this section, the grammer of the gstreamer command-line pipeline and some usefule debug tips are introduced. **They have been moved to the end of the article.**

## Pipeline samples

Basically, you neet to install `gstreamer1.0-plugins-base`, `gstreamer1.0-plugins-good`, `gstreamer1.0-plugins-bad`, `gstreamer1.0-omx-generic`, `gstreamer1.0-omx-bellagio-config`,`gstreamer1.0-tools`.

### Test

#### videotestsrc

```shell
# videotestsrc
gst-launch-1.0 videotestsrc ! autovideosink

# specify the video stream format
gst-launch-1.0 videotestsrc ! video/x-raw, format=NV12, width=960, height=540, framerate=60/1 ! autovideosink
```

#### fpsdisplaysink

```shell
# display framerate
gst-launch-1.0 videotestsrc ! fpsdisplaysink

# no need to sync on the clock - used to test the performance of the pipeline
gst-launch-1.0 videotestsrc ! fpsdisplaysink sync=false

# stop display on the screen, but redirect the output to stdout
gst-launch-1.0 videotestsrc ! fpsdisplaysink text-overlay=false -v 2>&1

# specify which sink to use
gst-launch-1.0 videotestsrc ! fpsdisplaysink sink=glimagesink

# combine the previous examples
gst-launch-1.0 videotestsrc ! fpsdisplaysink sink=glimagesink sync=false text-overlay=false -v 2>&1
```

#### audiotestsrc

```shell
# audiotestsrc
gst-launch-1.0 audiotestsrc ! autoaudiosink

# change volume (0 to 1)
gst-launch-1.0 audiotestsrc volume=0.1 ! autoaudiosink

# change waveform
# could be selected among square, silence, pink-noise, etc.
gst-launch-1.0 audiotestsrc wave=pink-noise ! autoaudiosink

# set fix frequency, such as "do" (262 Hz)
gst-launch-1.0 audiotestsrc freq=262 ! autoaudiosink
```

#### fakesink

```shell
# a dummy sink that swallows everything
gst-launch-1.0 videotestsrc ! fakesink
```

### Decode

#### Video decode

```shell
# let decodebin choose which decoder to use,
# and autovideosink choose which video-sink to use (not recommended)
gst-launch-1.0 filesrc location=fire.mp4 ! decodebin ! autovideosink

# h264 software decode without opengl display
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux ! h264parse ! avdec_h264 ! xvimagesink

# h264 hardware decode with opengl display
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux ! h264parse ! omxh264dec ! glimagesink

# h265 hardware decode
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux ! h265parse ! omxh265dec ! glimagesink

# vp9 hardware decode 
gst-launch-1.0 filesrc location=fire.webm ! matroskademux ! omxvp9dec ! glimagesink

# mpeg4 hardware decode 
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux ! queue ! mpeg4videoparse ! omxmpeg4videodec ! glimagesink

# play mkv/webm file
gst-launch-1.0 filesrc location=fire.mkv ! matroskademux ! decodebin ! glimagesink

# todo 10-bit h264/h265 decode
```

#### Audio decode

```shell
# there is no hardware decoder on th1520

# let autoaudiosink choose which audio-sink to use
gst-launch-1.0 filesrc location=blade.mp3 ! decodebin ! audioconvert ! autoaudiosink

# mp3 decode
gst-launch-1.0 filesrc location=blade.mp3 ! mpegaudioparse ! avdec_mp3 ! audioconvert ! autoaudiosink

# aac decode
gst-launch-1.0 filesrc location=blade.aac ! aacparse ! avdec_aac ! audioconvert ! autoaudiosink

# opus decode
## todo: opusparser stuck in prerolling stage
 gst-launch-1.0 filesrc location=blade.opus ! opusparse ! opusdec ! audioconvert ! autoaudiosink

# use specific audiosink
gst-launch-1.0 filesrc location=blade.mp3 ! decodebin ! audioconvert ! pulsesink

# specify the output device by using alsasink with device property
gst-launch-1.0 filesrc location=blade.mp3 ! decodebin ! audioconvert ! alsasink device=hw:0,2
```

#### demux and decode

```shell
# play mp4 file with both audio and video
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux name=demux \
  demux.video_0 ! queue  ! decodebin ! videoconvert ! autovideosink \
  demux.audio_0 ! queue ! decodebin ! audioconvert !  autoaudiosink
```

### Encode

#### Video encode

```shell
# h264 encode test
# before import video stream to omxh264dec, data should be transformed to frame with rawvideoparse
# property and pad of rawvideoparse should be set to the same
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse format=nv12 width=640 height=480 ! omxh264enc ! fakesink

# h264 hardware encode to file
## todo: encoded h264 file has seek problem
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse format=nv12 width=640 height=480 ! omxh264enc \
  ! h264parse ! qtmux ! mp4mux ! filesink location=test.mp4

# h264 hardware encode to file with specific bitrate(bit per second)
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse format=nv12 width=640 height=480  \
  ! omxh264enc target-bitrate=3000000 \
  ! h264parse ! filesink location=test.mp4


# h265 hardware encode to file
## this pipeline produces h265 stream only
## qtdemux is not needed while decoding
 gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse format=nv12 width=640 height=480 ! omxh265enc \
  ! h265parse ! filesink location=test.h265

# h264 hardware encode from camera to file
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert \
  ! video/x-raw, format=NV12,width=640,height=480 \
  ! rawvideoparse format=nv12 width=640 height=480 \
  ! omxh264enc ! h264parse ! filesink location=test.mp4 -e
```

#### Audio encode

```shell
# There is no hardware audio encoder on th1520

# todo: add software audio encode examples
```

#### mux and encode

```shell
# mux test
gst-launch-1.0 audiotestsrc ! autoaudiosink videotestsrc ! autovideosink

# todo: there is some problem with qtmux and mp4mux,
```

### Media transcode

### Video scalling

### Media mixing

### Camera capture

### Stream transfer

### Other tools

#### gst-inspect-1.0

`gst-inspec-1.0` is a tool to print info about a GStreamer plugin or element.

included in `gstreamer1.0-tools`.

#### gst-discover1.0

`gst-discover-1.0` is inclucded in `gstreamer1.0-plugins-base-apps`.

#### gst-play-1.0

### Other examples

## Get ready

### Debug tips

To get the cap info or property of an element, you may need to run `gst-inspect-1.0 <element_name>`. If `gst-inspect-1.0` command not found, install `gstreamer1.0-tools`.

If you want to debug GStreamer, refer to the articles below.
[https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html](https://gstreamer.freedesktop.org/documentation/tutorials/basic/debugging-tools.html)
[https://gstreamer.freedesktop.org/documentation/gstreamer/running.html?gi-language=c](https://gstreamer.freedesktop.org/documentation/gstreamer/running.html?gi-language=c)

### Grammer

Here is a simple command-line pipeline.

```shell
gst-launch-1.0 videotestsrc ! autovideosink
```

**`!` represent a link.** The established pipeline looks like

```plain
videotestsrc => autovideosink
```

There are several properties  in videotestsrc, which could be lookup by `gst-inspect-1.0 videotestsrc`.Set some properties:

```shell
gst-launch-1.0 videotestsrc pattern=ball flip=true ! autovideosink
```

**`NAME=VALUE` is a property.** `pattern` and `flip` are properties of videotestsrc.

Sometimes we need to control the behavior of `gst-launch-1.0`.

```shell
GST_DEBUG=3 gst-launch-1.0 videotestsrc pattern=ball flip=true ! autovideosink
gst-launch-1.0 videotestsrc pattern=ball flip=true ! autovideosink --gst-debug-level=3
```

There are two ways to achieve this goal. One is setting **environment variable**. Another one is passing **command-line argument** to `gst-launch-1.0`. `GST_DEBUG=3` and `--gst-debug-level=3` have the same meaning.

In most video decoding cases, there is a video-stream and an audio stream. We need to use **demuxer** to seperate them. Demuxer usually have several src pad. Here is an example.

```shell
gst-launch-1.0 filesrc location=test-video.mp4 ! qtdemux name=demux demux. ! queue ! h264parse ! omxh264dec ! glimagesink demux. ! queue ! aacparsaacparse ! avdec_aac ! audioconvert !  alsasink
```

It is hard to read. Add some word wrap:

```shell
gst-launch-1.0 filesrc location=test-video.mp4 ! qtdemux name=demux \
  demux. ! queue ! h264parse ! omxh264dec ! glimagesink \
  demux. ! queue ! aacparse ! avdec_aac ! audioconvert ! pulsesink
```

Here we use a demuxer to seperate video and audio, the grammer is:

```plain
... demuxer name=DEMUXER_NAME \
DEMUXER_NAME.PIPELINE_NAME_1 ! queue ! ...(the remaining part of pipeline 1) \
DEMUXER_NAME.PIPELINE_NAME_2 ! queue ! ...(the remaining part of pipeline 2)
```

Sometimes we need to set certain properties of the stream at certain nodes in the pipeline, e.g set resolution of the videostream of the `videotestsrc`.

```shell
gst-launch-1.0 videotestsrc ! video/x-raw, width=800, height=600 ! glimagesink
```

We negotiated the pad properties between `videotestsrc` and `glimagesink`. The property names and values must be supported by both element. You can find them in `Pad Templates` section of `gst-inspect-1.0`.

## Reference

1. [GStreamer pipeline grammar - StackOverflow](https://stackoverflow.com/questions/25382211/)
2. [GStreamer pipeline samples - GitHub](https://gist.github.com/hum4n0id/cda96fb07a34300cdb2c0e314c14df0a)
3. [gst-launch-1.0 - GStreamer doc](https://gstreamer.freedesktop.org/documentation/tools/gst-launch.html)
4. [How to display fps of streaming video in gsteramer? - StackOverflow](https://stackoverflow.com/questions/73948308/)
5. [Storing AAC Audio and Retrieving - StackOverflow](https://stackoverflow.com/questions/37496912)
6. [Accelerated GStreamer User Guide - NVIDIA](https://developer.download.nvidia.com/embedded/L4T/r32_Release_v1.0/Docs/Accelerated_GStreamer_User_Guide.pdf)
