
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
gst-launch-1.0 filesrc location=blade.aac ! aacparse ! faad ! audioconvert ! autoaudiosink
## faad works well without aacparse
gst-launch-1.0 filesrc location=blade.aac ! faad ! audioconvert ! autoaudiosink

# opus decode
## ogg file must be demuxed by oggdemux first
gst-launch-1.0 filesrc location=blade.ogg ! oggdemux ! opusparse ! opusdec ! audioconvert ! autoaudiosink
gst-launch-1.0 filesrc location=blade.ogg ! oggdemux ! opusparse ! avdec_opus ! audioconvert ! autoaudiosink

# wav decode
gst-launch-1.0 filesrc location=test.wav ! wavparse ! audioresample ! audioconvert ! autoaudiosink

# use specific audiosink
gst-launch-1.0 filesrc location=blade.mp3 ! decodebin ! audioconvert ! pulsesink

# specify the output device by using alsasink with device property
gst-launch-1.0 filesrc location=blade.mp3 ! decodebin ! audioconvert ! alsasink device=hw:0,2
```

#### demux and decode

We play media file in this section.

```shell
# play mp4 file with both audio and video
gst-launch-1.0 filesrc location=fire.mp4 ! qtdemux name=demux \
  demux.video_0 ! queue  ! decodebin ! videoconvert ! autovideosink \
  demux.audio_0 ! queue ! decodebin ! audioconvert !  autoaudiosink
```

### Encode to file

#### Video encode

```shell
# h264 encode test
# before import video stream to omxh264dec, data should be transformed to frame with rawvideoparse
# property and pad of rawvideoparse should be set to the same, so we use 'use-sink-caps=true' here
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse use-sink-caps=true ! omxh264enc ! fakesink

# h264 hardware encode to file
## todo: encoded h264 file has seek problem
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse use-sink-caps=true ! omxh264enc \
  ! h264parse ! qtmux ! mp4mux ! filesink location=test.mp4

# h264 hardware encode to file with specific bitrate(bit per second)
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse use-sink-caps=true  \
  ! omxh264enc target-bitrate=3000000 \
  ! h264parse ! filesink location=test.mp4


# h265 hardware encode to file
## this pipeline produces h265 stream only
## qtdemux is not needed while decoding
gst-launch-1.0 videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=640, height=480 \
  ! rawvideoparse use-sink-caps=true ! omxh265enc \
  ! h265parse ! filesink location=test.h265

# h264 hardware encode from camera to file
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert \
  ! video/x-raw, format=NV12,width=640,height=480 \
  ! rawvideoparse use-sink-caps=true \
  ! omxh264enc ! h264parse ! filesink location=test.mp4 -e
```

#### Audio encode

```shell
# There is no hardware audio encoder on th1520

# encode aac stream with adts container(.aac file)
## the unit of the bitrate is 'bit/sec'
gst-launch-1.0 audiotestsrc ! voaacenc bitrate=128000 ! avmux_adts ! filesink location=test.aac


# todo: encode aac stream with adif container(.m4a file)

# encode to mp3 file
## the unit of the bitrate is 'kbit/sec'
gst-launch-1.0 audiotestsrc ! lamemp3enc quality=2 target=bitrate bitrate=192 cbr=true ! id3v2mux ! filesink location=test.mp3

# encode opus stream to .ogg file
gst-launch-1.0 audiotestsrc ! opusenc ! oggmux ! filesink location=test.opus


# todo: encode pcm stream to .wav file
```

#### mux and encode

This part has been removed to `Video + audio transcode` section.

### Media file transcode

#### Video transcode

Since `omx...dec` cannot fulfill qtdemux and mp4mux, and gst-omx is no longer maintained, it is hard to mux mp4 file with these two plugins. To mux stream from omxh264dec, use `avmux_mp4` instead. But there is no h265 or vp9 muxer available for `omx...dec`

The raw video stream should be processed by `rawvideoparse` before sent to encoder. Because  itself cannot scale frame or change framerate, `rawvideoparse` need to get the stream info of its sink pad(src pad of the backward element). Therefore `use-sink-caps` must be set to `true`.

To change width and height, set properties `output-width` and `output-height` of `omx...dec`. To modify bitrate and bitrate control method, set properties `control-rate` and `target-bitrate` of `omx...enc`. To change framerate, set properties of `videorate`.

```shell
# h265 to h264
gst-launch-1.0 filesrc location=test_h265.mp4 ! qtdemux ! h265parse ! omxh265dec \
  ! rawvideoparse use-sink-caps=true \
  ! omxh264enc ! h264parse ! avmux_mp4 ! filesink location=t_h264.mp4

# vp9 to h264
gst-launch-1.0 filesrc location=test_vp9.webm ! matroskademux ! omxvp9dec \
  ! rawvideoparse use-sink-caps=true \
  ! omxh264enc ! h264parse ! avmux_mp4 ! filesink location=t_h264.mp4

# arbitrary input to h264
gst-launch-1.0 filesrc location=test_h264.mp4 ! decodebin \
  ! rawvideoparse use-sink-caps=true \
  ! omxh264enc ! h264parse ! filesink location=t_h264.mp4

# h264 to h264, with more options
## set the video width and height to 1280×720, framerate to 15fps, bitrate mode to constant and bitrate to 5Mbps
gst-launch-1.0 filesrc location=test_h264.mp4 ! qtdemux ! h264parse \
  ! omxh264dec output-width=1280 output-height=720 \
  ! videorate ! video/x-raw, framerate=15/1 \
  ! rawvideoparse use-sink-caps=true  \
  ! omxh264enc control-rate=constant target-bitrate=5000000 ! h264parse ! filesink location=t_h264.mp4

## there is no vp9 encoder in th1520 omxil lib
```

#### Audio transcode

```shell
# aac to mp3
gst-launch-1.0 filesrc location=test.aac ! aacparse ! avdec_aac ! audioconvert ! lamemp3enc quality=2 target=bitrate bitrate=192 cbr=true ! id3v2mux ! filesink location=t.mp3

# mp3 to aac
gst-launch-1.0 filesrc location=test.mp3 ! mpegaudioparse ! avdec_mp3 ! audioconvert ! voaacenc bitrate=128000 ! avmux_adts ! filesink location=t.aac

# wav to mp3
gst-launch-1.0 filesrc location=test.wav ! wavparse ! audioresample ! audioconvert ! lamemp3enc quality=2 target=bitrate bitrate=192 cbr=true ! id3v2mux ! filesink location=t.mp3

# mp3 to wav
gst-launch-1.0 filesrc location=test.mp3 ! mpegaudioparse ! avdec_mp3 ! audioresample ! audioconvert \
  ! audio/x-raw, rate=44100, format=S16LE ! wavenc ! filesink location=t.wav
```

#### Video + audio remux and transcode

```shell
# mux test
gst-launch-1.0 audiotestsrc ! autoaudiosink videotestsrc ! autovideosink

# mux the test video and audio stream to a mp4 file
## be aware that '-e' must be set to this pipeline,
## and there is no '!' before audiotestsrc
gst-launch-1.0 -e videotestsrc ! videoconvert \
  ! video/x-raw, format=NV12, width=960, height=540 \
  ! rawvideoparse use-sink-caps=true ! omxh264enc ! h264parse \
  ! avmux_mp4 name=mux ! filesink location=t_h264.mp4 \
  audiotestsrc ! lamemp3enc ! mux.

# change container from mkv to mp4 with h264 stream
## this means demux a mkv file then mux video and audio stream to mp4 file
gst-launch-1.0 filesrc location=test_h264.mkv \
  ! matroskademux name=demux mp4mux force-create-timecode-trak=true name=mux ! filesink location=t_h264.mp4 \
  demux.video_0 ! queue ! video/x-h264 ! mux. \
  demux.audio_0 ! queue ! audio/mpeg ! mux.
```

### Media mixing

### Camera capture

You can use command `v4l2-ctl`, which is included in package `v4l-utils` to get information of your camera.

For more information, read [this article](https://archive.md/nlyBK)

```shell
# h264 hardware encode from camera to file
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert \
  ! video/x-raw, format=NV12,width=640,height=480 \
  ! rawvideoparse format=nv12 width=640 height=480 \
  ! omxh264enc ! h264parse ! filesink location=test.mp4 -e

```

### Stream transfer

```bash
# capture camera and stream to other machine

## source
gst-launch-1.0 v4l2src device=/dev/video0 !  videoconvert ! videorate \
  ! video/x-raw,  format=NV12,width=640,height=480,framerate=20/1 \
  ! rawvideoparse format=nv12 width=640 height=480  framerate=20/1 \
  ! omxh264enc ! h264parse config-interval=1 ! video/x-h264,stream-format=byte-stream,alignment=nal \
  ! rtph264pay ! udpsink  host=192.168.31.27 port=5600

## destination
gst-launch-1.0 udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' \
  ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink
```

### Other tools

#### gst-inspect-1.0

`gst-inspect-1.0` is a tool to print info about a GStreamer element(factory), which is included in `gstreamer1.0-tools`.

```shell
# print the GStreamer element list with a 'less' like paing filter.
gst-inspect-1.0

# print info of the element
gst-inspect-1.0 <element_name>
```

#### gst-discover1.0

`gst-discover-1.0` is a tool to show info about the media file, which is inclucded in `gstreamer1.0-plugins-base-apps`.

```shell
gst-discoverer-1.0 -v test.mp4
```

#### gst-play-1.0

If you are tired of manually build pipeline for playback by hand. You can use `gst-play-1.0` as an alternative, which is included in `gstreamer1.0-plugins-base-apps`.

```shell
# play media file
gst-play-1.0 test.mp4

# play media file with specific sink
gst-play-1.0 --videosink=glimagesink --audiosink=alsasink
```

Left button and right button can be used to seek. For more info, please read [this article](https://manpages.ubuntu.com/manpages/xenial/man1/gst-play-1.0.1.html).

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
gst-launch-1.0 filesrc location=test-video.mp4 ! qtdemux name=demux demux. ! queue ! h264parse ! omxh264dec ! glimagesink demux. ! queue ! aacparse ! avdec_aac ! audioconvert !  alsasink
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
7. [Storing AAC Audio and Retrieving - StackOverflow](https://stackoverflow.com/questions/37496912)
8. [Play an opus file with gstreamer and pulseaudio - StackOverflow](https://stackoverflow.com/questions/70672729)
9. [mp4mux not working with omxh264enc](https://forums.raspberrypi.com/viewtopic.php?t=206714&sid=ec650e96fc653a8ad0e06dd099cb9220)
10. [omxh264enc makes qtmux error out with "Buffer has no PTS." - FreeDesktop - Gitlab](https://gitlab.freedesktop.org/gstreamer/gst-omx/-/issues/13)
11. [gst-omx: Retire the whole package - FreeDesktop - Gitlab](https://gitlab.freedesktop.org/gstreamer/gstreamer/-/merge_requests/4976/)
