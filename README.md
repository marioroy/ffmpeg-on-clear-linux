## ffmpeg-on-clear-linux

Run [FFmpeg](https://ffmpeg.org/) on [Clear Linux](https://clearlinux.org/) including H.264 and VP9 playback on the GPU.

* [What's included](#whats-included)
* [Initial requirements and preparation](#requirements)
* [Building and installation](#building)
* [Multiple libs in x264 and x265](#multiple-libs)
* [Determining the VA-API driver to use](#va-api-driver)
* [Firefox config file and settings](#firefox)
* [Chromium and others installation](#chromium-and-others)
* [High DPI support](#high-dpi-support)
* [How can I make sure hardware acceleration is working?](#verify-acceleration)
* [Watch HDR content](#watch-hdr-content)
* [See also](#advert)

Thank you, @xtknight for the initial [VP9](https://github.com/xtknight/vdpau-va-driver-vp9) acceleration bits. Likewise and thank you, @xuanruiqi for the [VP9-update](https://github.com/xuanruiqi/vdpau-va-driver-vp9) to include additional fixes. Finally, thank you @elFarto for the [NVDEC-enabled](https://github.com/elFarto/nvidia-vaapi-driver) driver. Both drivers can co-exist with few tweaks to the installation process.

### <a id="whats-included">What's included

This is an automation **how-to** for building FFmpeg and minimum dependencies. It builds two VA-API drivers for use with NVIDIA graphics.

```text
bin        Browser launch scripts to be copied to $HOME/bin/.
build-all  Top-level script for building dependencies and FFmpeg.
desktop    Desktop files for $HOME/.local/share/applications/.
doc        Browser configuration and installation guides.
extras     Complementary YouTube player for testing nvdec/nvenc.
localenv   Set your NVIDIA GPU's CUDA compute capability here.
scripts    Contains individual build-install scripts.
```

### <a id="requirements">Initial requirements and preparation

Important, see [guide](doc/NV-Requirements-And-Preparation.md) in doc folder if using the NVIDIA proprietary driver. For CUDA support in FFmpeg, be sure to install the `c-extras-gcc10` bundle if the base `gcc --version` is later than 10.

### <a id="building">Building and installation

The build and installation process is fully automatic.

```bash
git clone https://github.com/marioroy/ffmpeg-on-clear-linux
cd ffmpeg-on-clear-linux
sudo bash build-all
```

Or become root and run each script individually. If choosing this path, be sure to run `000-install-dependencies` first. Various scripts exit silently depending on whether `/opt/nvidia/bin/nvidia-settings` or `/usr/local/cuda/cuda.h` exists.

```bash
sudo root
cd scripts

./000-install-dependencies
./100-build-nv-codec-headers
./110-build-libvdpau
...
```

The `builddir` (once created) serves as a cache folder. Remove the correspondent `*.tar.gz` file(s) to re-fetch or re-clone from the internet.

I'm hoping that the build process succeeds for you as it does for me. However, I may have a bundle installed that's missing in `000-install-dependencies`. Please reach out if that's the case. The Media SDK is included in the FFmpeg build for Intel hardware, but not yet tested what else is needed for that platform.

Remember to add `/usr/local/bin` to your `PATH` environment variable if not already done, preferably before `/usr/bin`.

### <a id="multiple-libs">Multiple libs in x264 and x265

The build scripts build each library independently. Below you will find that `x264` supports 8-bits and 10-bits output.

```bash
$ x264 --help | grep "Output bit depth"
Output bit depth: 8/10

$ ffmpeg -hide_banner -h encoder=libx264 | grep "Supported pixel formats"
Supported pixel formats: yuv420p yuvj420p yuv422p yuvj422p yuv444p yuvj444p
nv12 nv16 nv21 yuv420p10le yuv422p10le yuv444p10le nv20le gray gray10le
```

The `x264` binary is not compiled with `lavf` (i.e. libavformat part of FFmpeg), `avs`, and `swscale` support as that would introduce a circular dependency. Instead, please refer to FFmpeg's [bindings](https://ffmpeg.org/ffmpeg-all.html#libx264_002c-libx264rgb) for libx264 options. You can specify an `x264` option with `-x264-params` in case there's no native FFmpeg binding available. See also, FFmpeg's beginner-friendly [guide](https://trac.ffmpeg.org/wiki/Encode/H.264) for H.264.

Similarly x265 supports 8-bits, 10-bits and 12-bits output.

```bash
$ x265 --help | grep "Output bit depth"
-D/--output-depth 8|10|12    Output bit depth... Default 8

$ ffmpeg -hide_banner -h encoder=libx265 | grep "Supported pixel formats"
Supported pixel formats: yuv420p yuvj420p yuv422p yuvj422p yuv444p yuvj444p
gbrp yuv420p10le yuv422p10le yuv444p10le gbrp10le yuv420p12le yuv422p12le
yuv444p12le gbrp12le gray gray10le gray12le
```

Please refer to FFmpeg's [bindings](https://ffmpeg.org/ffmpeg-all.html#libx265) for libx265 options. You can specify an `x265` option with `-x265-params`. See also, FFmpeg's beginner-friendly [guide](https://trac.ffmpeg.org/wiki/Encode/H.265) for H.265.

### <a id="va-api-driver">Determining the VA-API driver to use

For hardware acceleration to work, the browser may have the driver built-in or you will need a suitable driver, i.e. `ls /usr/lib64/dri/*_drv_video.so`. To be sure, run `vainfo` in a terminal window. For AMD try `LIBVA_DRIVER_NAME=r600 vainfo` or `LIBVA_DRIVER_NAME=radeonsi vainfo`. For Intel the `iHD` driver is newer. So check first `LIBVA_DRIVER_NAME=iHD vainfo` or try `LIBVA_DRIVER_NAME=i965 vainfo`.

Below see captured output for the NVIDIA driver.

```bash
$ LIBVA_DRIVER_NAME=nvidia vainfo   # nvidia is a symbolic link to vdpau
$ LIBVA_DRIVER_NAME=vdpau  vainfo

libva info: VA-API version 1.13.0
libva info: User environment variable requested driver 'vdpau'
libva info: Trying to open /usr/lib64/dri/vdpau_drv_video.so
libva info: Found init function __vaDriverInit_1_13
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.13 (libva 2.13.0)
vainfo: Driver version: Splitted-Desktop Systems VDPAU backend for VA-API - 0.7.4
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG4Simple            : VAEntrypointVLD
      VAProfileMPEG4AdvancedSimple    : VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointVLD
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileVP9Profile0            : VAEntrypointVLD
```

### <a id="firefox">Firefox config file and settings

See [guide](doc/Firefox-Config-File-And-Settings.md) in doc folder. Be sure to review and compare settings in Firefox, particularly `media.rdd-process.enabled`. Leave enabled or the NVDEC-enabled VA-API driver will not work.

### <a id="chromium-and-others">Chromium and others installation

See [guide](doc/Chromium-And-Others-Installation.md) in doc folder. This covers Brave, Chromium, Google Chrome, and Vivaldi.

### <a id="high-dpi-support">High DPI Support

First, run gnome-tweaks and adjust "Fonts" > "Scaling Factor". Enter a floating value or press the `+` or `-` buttons until reaching the screen DPI divided by 96. For example, a 109 DPI screen divided by 96 equals 1.14 for the scaling factor rounded to 2 decimal places. That will update the `Xft.dpi` value, preferably matching the screen DPI. Subsequently, adjust the font size to 11 or 10 for "Interface Text", "Document Text", and "Legacy Window Titles"; size 13 or 12 for Monospace Text.

```bash
gnome-tweaks                       # as normal user
xrdb -query                        # Xft.dpi: 109
```

The launch scripts for Chromium-based browsers set the scale-factor automatically, based on the `Xft.dpi` value. You may find that the right edge of the window is not straight all the way to the top. Simply edit the `~/bin/run-*` launch script and adjust the width in 2 pixels increment until a straight edge.

For Firefox, go to "about:config" and change the `layout.css.devPixelsPerPx` value manually. Start with 1.0 and increase-decrease in 0.01 increments or enter the value for `Xft.dpi` divided by 96. For example, 109 DPI / 96 = 1.135416667.

```text
layout.css.devPixelsPerPx          1.135416667
```

Firefox context-menus may not work in Wayland. Ensure the context-menu is working by right-clicking in the URL field. If the popup-menu is not sticking, then go to `about:config` and try decreasing or increasing the value (i.e. 1.0, 1.12, 1.25, 1.5, 2.0).

```text
layout.css.devPixelsPerPx          1.12
```

### <a id="verify-acceleration">How can I make sure hardware acceleration is working?

In Firefox, check the `about:support` page. In Brave, Chromium, Google Chrome, and Vivaldi, check the `chrome://gpu` page. Another way is running a utility suited for your hardware while watching a video. The `igt-gpu-tools` bundle provides `intel_gpu_top` and `intel_gpu_frequency`.

```bash
sudo swupd bundle-add igt-gpu-tools
```

1. `watch -n 1 /opt/nvidia/bin/nvidia-smi` to check if "GPU-Util" percentage goes up
2. `sudo intel_gpu_top` to check if percentage under the "Video" section goes up
3. `watch -n 1 sudo intel_gpu_frequency` to check if the frequency goes up

Depending on the quality of the video (1080p60 or lesser), the video codec may sometimes not decode on the GPU. For example, AV1 codec. A workaround is to try installing the `enhanced-h264ify` extension to make YouTube stream H.264 videos instead, but also allow VP8/VP9 via the extension settings. To disable AV1 altogether; in Firefox, go to `about:config` and set `media.av1.enabled` to `false`. That will fall back to using another codec such as VP9. Install the `Not yet, AV1` extension for Google Chrome and like browsers.

### <a id="watch-hdr-content">Watch HDR content

To play HDR content, see `youtube-play` inside the extras folder.

### <a id="advert">See also, advert at Clear Linux

* [Annoucement and benchmark results](https://community.clearlinux.org/t/ffmpeg-supporting-h-264-and-vp9-hardware-acceleration-in-firefox/6148)

