## firefox-config-file-and-settings

* [Firefox config file](#config)
* [Review Firefox settings](#settings)
* [See also](#wikis)

### <a id="config">Firefox config file

The following is my Firefox config. Update the value for `LIBVA_DRIVER_NAME` or leave it `auto`. Subsequently, the driver name is overridden automatically for NVIDIA hardware.

```bash
$ cat ~/.config/firefox.conf

export FONTCONFIG_PATH=/usr/share/defaults/fonts
export LD_LIBRARY_PATH=/usr/local/lib
export LIBVA_DRIVERS_PATH=/usr/lib64/dri:/usr/local/lib/dri

# Optionally uncomment one of the following matching your configuration.
# export LIBVA_DRIVER_NAME=iHD
# export LIBVA_DRIVER_NAME=i965
# export LIBVA_DRIVER_NAME=r600
# export LIBVA_DRIVER_NAME=radeonsi

if [[ -d /opt/nvidia ]]
then
    # Add /opt/nvidia/{lib64,lib} to path.
    export LD_LIBRARY_PATH="/opt/nvidia/lib64:/opt/nvidia/lib:$LD_LIBRARY_PATH"

    # The NVDEC-backend driver works in x11 and Wayland.
    if [[ -f /usr/local/lib/dri/nvdec_drv_video.so ]]; then
        export LIBVA_DRIVER_NAME=nvdec
    else
        export LIBVA_DRIVER_NAME=vdpau
    fi
fi

if [[ $XDG_SESSION_TYPE == wayland ]]
then
    export EGL_PLATFORM=wayland
    export MOZ_ENABLE_WAYLAND=1
else
    export MOZ_DISABLE_WAYLAND=1
    export MOZ_X11_EGL=1
fi

export MOZ_ACCELERATED=1
export MOZ_DISABLE_RDD_SANDBOX=1
export MOZ_USE_XINPUT2=1
export MOZ_WEBRENDER=1
```

### <a id="settings">Review Firefox settings

Below are the minimum settings applied via `about:config` to enable hardware acceleration. The `media.rdd-ffmpeg.enable` flag must be enabled for h264ify or enhanced-h264ify to work along with VP9. Basically, this allows you to choose to play videos via the h264ify extension or VP9 media by disabling h264ify and enjoy beyond 1080P playback.

```text
gfx.canvas.azure.accelerated                   true
gfx.webrender.all                              true
gfx.webrender.enabled                          true

Enable software render if you want to render on the CPU instead of GPU.
Preferably, leave this setting false since webrender on the GPU is needed
to decode videos in hardware.
gfx.webrender.software                         false

Do not add xrender if missing or set to false or click on the trash icon.
This is a legacy setting that shouldn't be used as it disables WebRender.
gfx.xrender.enabled                            false

Ensure false so to be on a supported code path for using WebRender.
layers.acceleration.force-enabled              false

media.ffmpeg.dmabuf-textures.enabled           true
media.ffmpeg.vaapi-drm-display.enabled         true
media.ffmpeg.vaapi.enabled                     true
media.ffvpx.enabled                            false

Verify enabled, necessary for the NVIDIA-NVDEC enabled driver to work.
media.rdd-ffmpeg.enabled                       true
media.rdd-process.enabled                      true

Optionally enable for NVIDIA 3000+ series graphics using proprietary
driver v510+ and NVDEC-enabled VA-API driver v0.0.5+. Disable if AV1
content does not appear to be hardware-accelerated.
media.av1.enabled                              false

Enable for Intel graphics supporting AV1 decoding.
media.av1.enabled                              true

Enable FFMPEG VA-API decoding support for WebRTC on Linux.
media.navigator.mediadatadecoder_vpx_enabled   true

Enable to help get decoding to work for NVIDIA 470 driver series.
widget.dmabuf.force-enabled                    true
```

### <a id="wikis">See also, wikis at Arch Linux

* [Firefox](https://wiki.archlinux.org/title/Firefox)
* [Hardware video acceleration](https://wiki.archlinux.org/title/Hardware_video_acceleration)

