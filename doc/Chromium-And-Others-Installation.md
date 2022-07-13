## chromium-and-others-installation

* [Chromium installation and configuration](#chromium)
* [Google Chrome installation and run script](#google-chrome)
* [Vivaldi installation and run script](#vivaldi)
* [Brave installation and run script](#brave)
* [Caveat with RPM package installation](#caveat)
* [See also](#wikis)

### <a id="chromium">Chromium installation and configuration

[Chromium](https://dev.chromium.org/Home) is an open-source browser project. Some say it's a browser made for developers. The [chromium-latest-linux](https://github.com/scheib/chromium-latest-linux) repository works great for launching Chromium including VP9 media playback.

**Installation**

Change directory to your home directory. The launch script for Chromium will look for the folder here. Run the `update.sh` script initially and periodically to fetch the latest snapshot. The other scripts `update-and-run.sh` and `run-chrome.sh` are not used.

```bash
$ pushd $HOME
$ git clone https://github.com/scheib/chromium-latest-linux.git
$ cd chromium-latest-linux
$ ./update.sh
$ popd
```

**Edit ~/bin/run-chromium-latest**

First copy the custom launch script to your `bin` folder.

```bash
$ mkdir -p ~/bin
$ cp ~/Downloads/ffmpeg-on-clear-linux/bin/run-chromium-latest ~/bin/.
```

Scroll down towards the end of the file. Update the value for `LIBVA_DRIVER_NAME` or leave it `auto`. The driver name is overridden automatically for NVIDIA hardware.

Opening new windows may be larger than the initial window. After a while, that can be annoying. The `--window-size=x,y` option resolves this issue. Optionally adjust the width and height (in pixels) appropriate for your display.

Accelerated 2D canvas is required (default enabled) to decode videos on the GPU. Two more options `--use-gl` and `--enable-features=VaapiVideoDecoder` are needed for hardware acceleration when watching a video. Hardware acceleration stopped working in Google Chrome 98. The `--disable-features=UseChromeOSDirectVideoDecoder` option resolves the issue by decoding videos using VDAVideoDecoder.

```bash
# Launch browser.
export FONTCONFIG_PATH=/usr/share/defaults/fonts
export LIBVA_DRIVERS_PATH=/usr/lib64/dri
export LIBVA_DRIVER_NAME=auto

if [[ -d /opt/nvidia && -f $LIBVA_DRIVERS_PATH/vdpau_drv_video.so ]]
then
    # Add /opt/nvidia/{lib64,lib} to path.
    export LD_LIBRARY_PATH="/opt/nvidia/lib64:/opt/nvidia/lib"

    # The VDPAU-backend driver works in x11 only.
    export LIBVA_DRIVER_NAME=vdpau
fi

[[ $XDG_SESSION_TYPE == wayland ]] && GL=egl || GL=desktop

exec "$EXECCMD" --window-size=1214,1000 \
    --disable-features=UseChromeOSDirectVideoDecoder \
    --disable-font-subpixel-positioning --disable-gpu-vsync \
    --disable-gpu-driver-bug-workarounds --enable-zero-copy \
    --enable-accelerated-2d-canvas --enable-smooth-scrolling \
    --enable-features=VaapiVideoDecoder,CanvasOopRasterization \
    --enable-gpu-rasterization --use-gl=$GL \
    --user-data-dir="$DATADIR" $* &> /dev/null &
```

**Running**

On first launch go into `Settings -> Appearance -> Customize fonts` and change the fonts. Metrically compatible with `Times New Roman`, `Arial`, and `Courier New` are `Tinos`, `Arimo`, and `Cousine` respectively. Optionally go into `Settings -> Advanced -> System` and disable "Use hardware acceleration when available" if the GPU is lacking or you prefer the CPU to decode videos.

A desktop file is created the first time it is run and placed in `~/.local/share/applications`. You may run Chromium using the command-line or search for "Chromium" in Application Finder. Launching from the desktop will run the same script.

```bash
$ ~/bin/run-chromium-latest
```

### <a id="google-chrome">Google Chrome installation and run script

[Google Chrome](https://www.google.com/chrome/) is a browser built by Google. You will find that the browser is quite fast. For NVIDIA graphics, video playback utilizes the Video Engine.

**Installation**

The `RPM` file for Google Chrome can be found at [Google](https://www.google.com/chrome/) and [pkgs.org](https://pkgs.org/download/google-chrome) (stable under RPM Packages).

**Note:** Installing Google Chrome will add the Google repository so your system will automatically keep Google Chrome up to date. If you don't want Google's repository (which is what we want), do `sudo touch /etc/default/google-chrome` before installing the package. The reason is the package will fail auto-install without the `--nodeps` flag.

The `-U` flag to `rpm` installs the new package, otherwise upgrades the installed package. Periodically, obtain the current stable release and run the `rpm` command as shown.

```bash
$ sudo mkdir -p /etc/default && sudo touch /etc/default/google-chrome

# install-or-update package from Google
$ sudo rpm -Uvh --nodeps \
    ~/Downloads/google-chrome-stable_current_x86_64.rpm 2>/dev/null

# install-or-update package from pkgs.org, change version accordingly
$ sudo rpm -Uvh --nodeps \
    ~/Downloads/google-chrome-stable-103.0.5060.114-1.x86_64.rpm 2>/dev/null
```

**Edit ~/bin/run-chrome-stable**

Copy the launch script and corresponding desktop file. Refer to the notes above for editing the script i.e. `LIBVA_DRIVER_NAME`, et al.

```bash
$ mkdir -p ~/bin && mkdir -p ~/.local/share/applications

$ cp ~/Downloads/ffmpeg-on-clear-linux/bin/run-chrome-stable ~/bin/.
$ cp ~/Downloads/ffmpeg-on-clear-linux/desktop/google-chrome.desktop \
       ~/.local/share/applications/.
```

**Running**

On first launch (just like with Chromium), you may want to go into `Settings -> Appearance -> Customize fonts` and change the fonts. Metrically compatible with `Times New Roman`, `Arial`, and `Courier New` are `Tinos`, `Arimo`, and `Cousine` respectively. Like with Chromium, optionally go into `Settings -> Advanced -> System` and disable "Use hardware acceleration when available" if the GPU is lacking or you prefer the CPU to decode videos.

Run Chrome using the command-line or search for "Google Chrome" in Application Finder.

```bash
$ ~/bin/run-chrome-stable
```

### <a id="vivaldi">Vivaldi installation and run script

[Vivaldi](https://vivaldi.com) is yet another open-source browser. The main highlight is being able to communicate in a much more organized way, while keeping control of your data. That sounds delightful! Similarly to Google Chrome, this also utilizes the Video Engine while watching VP9 media.

**Installation**

The `RPM` file for Vivaldi can be found at [Vivaldi](https://vivaldi.com/download/). Download the Linux RPM 64bit package.

**Note:** Installing Vivaldi will add the Vivaldi repository so your system will automatically keep Vivaldi up to date. If you don't want Vivaldi's repository (which is what we want), do `sudo touch /etc/default/vivaldi` before installing the package. The reason is the package will fail auto-install without the `--nodeps` flag.

The `-U` flag to `rpm` installs the new package, otherwise upgrades the installed package. Periodically, obtain the current stable release and run the `rpm` command as shown.

```bash
$ sudo mkdir -p /etc/default && sudo touch /etc/default/vivaldi

# install-or-update package, change version accordingly
$ sudo rpm -Uvh --nodeps \
    ~/Downloads/vivaldi-stable-5.3.2679.68-1.x86_64.rpm 2>/dev/null
```

**Edit ~/bin/run-vivaldi-stable**

Copy the launch script and corresponding desktop file. See Chromium section above for editing the script i.e. `LIBVA_DRIVER_NAME`, et al.

```bash
$ mkdir -p ~/bin && mkdir -p ~/.local/share/applications

$ cp ~/Downloads/ffmpeg-on-clear-linux/bin/run-vivaldi-stable ~/bin/.
$ cp ~/Downloads/ffmpeg-on-clear-linux/desktop/vivaldi-stable.desktop \
       ~/.local/share/applications/.
```

**Running**

On first launch go into `Settings -> Webpages -> Fonts -> Default Fonts` and change the default fonts. Metrically compatible with `Times New Roman`, `Arial`, and `Courier New` are `Tinos`, `Arimo`, and `Cousine` respectively. Optionally go into `Settings -> Webpages` and uncheck "Use Hardware Acceleration When Available" if the GPU is lacking or you prefer the CPU to decode videos.

Run Vivaldi using the command-line or search for "Vivaldi" in Application Finder.

```bash
$ ~/bin/run-vivaldi-stable
```

### <a id="brave">Brave installation and run script

[Brave](https://brave.com) is an open-source browser, reimagined. It claims three times faster than Google Chrome and better privacy than Firefox. Similarly to Google Chrome and Vivaldi, this too utilizes the Video Engine while watching VP9 media.

**Installation**

The `RPM` file for Brave can be found at [sourceforge.net](https://sourceforge.net/projects/brave-browser.mirror/files/). Go to [pkgs.org](https://pkgs.org/download/brave) and scroll to the bottom of the page. It will mention the current release version.

**Note:** Installing Brave will add the Brave repository so your system will automatically keep Brave up to date. If you don't want Brave's repository (which is what we want), do `sudo touch /etc/default/brave-browser` before installing the package. The reason is the package will fail auto-install without the `--nodeps` flag.

The `-U` flag to `rpm` installs the new package, otherwise upgrades the installed package. Periodically, obtain the current stable release and run the `rpm` command as shown.

```bash
$ sudo mkdir -p /etc/default && sudo touch /etc/default/brave-browser

# install-or-update package, change version accordingly
$ sudo rpm -Uvh --nodeps \
    ~/Downloads/brave-browser-1.41.96-1.x86_64.rpm 2>/dev/null
```

**Edit ~/bin/run-brave-stable**

Copy the launch script and corresponding desktop file. See Chromium section above for editing the script i.e. `LIBVA_DRIVER_NAME`, et al.

```bash
$ mkdir -p ~/bin && mkdir -p ~/.local/share/applications

$ cp ~/Downloads/ffmpeg-on-clear-linux/bin/run-brave-stable ~/bin/.
$ cp ~/Downloads/ffmpeg-on-clear-linux/desktop/brave-browser.desktop \
       ~/.local/share/applications/.
```

**Running**

On first launch go into `Settings -> Appearance -> Customize fonts` and change the fonts. Metrically compatible with `Times New Roman`, `Arial`, and `Courier New` are `Tinos`, `Arimo`, and `Cousine` respectively. Like with other Chromium-based browsers, optionally go into `Settings -> Additional settings -> System` and disable "Use hardware acceleration when available" if the GPU is lacking or you prefer the CPU to decode videos.

Run Brave using the command-line or search for "Brave Web Browser" in Application Finder.

```bash
$ ~/bin/run-brave-stable
```

### <a id="caveat">Caveat with RPM package installation

It feels hacky in Clear Linux installing a package that was built for another platform such as RedHat. For peace of mind, check for missing library dependencies using the `ldd` utility. Ensure nothing is missing in the output. If true, then install missing packages with `sudo swupd bundle-add PKGNAME`. Run `sudo swupd search LIBNAME` if needed.

Another solution is building from source. This is likely not necessary, although becomes reality if unable to meet library dependencies. Uninstall the browser with `sudo rpm -e NAME`, given below.

```bash
$ ldd ~/chromium-latest-linux/latest/chrome 2>/dev/null | grep "not found$"
$ ldd /opt/brave.com/brave/brave 2>/dev/null | grep "not found$"
$ ldd /opt/google/chrome/chrome 2>/dev/null | grep "not found$"
$ ldd /opt/vivaldi/vivaldi-bin 2>/dev/null | grep "not found$"
```

Hackiness put aside, a benefit of using a package for installation is that the package can be uninstalled easily. Optionally remove your browser data and settings. Though, be sure to export your bookmarks.

```bash
# Brave
$ sudo rpm -e brave-browser 2>/dev/null
$ sudo rm -f /etc/default/brave-browser
$ rm -f ~/.local/share/applications/brave-browser.desktop
$ rm -fr ~/.cache/BraveSoftware/Brave-Browser
$ rm -fr ~/.config/BraveSoftware/Brave-Browser  (optional)
```

```bash
# Google Chrome
$ sudo rpm -e google-chrome-stable 2>/dev/null
$ sudo rm -f /etc/default/google-chrome
$ rm -f ~/.local/share/applications/google-chrome.desktop
$ rm -fr ~/.cache/google-chrome
$ rm -fr ~/.config/google-chrome  (optional)
```

```bash
# Vivaldi
$ sudo rpm -e vivaldi-stable 2>/dev/null
$ sudo rm -f /etc/default/vivaldi
$ rm -f ~/.local/share/applications/vivaldi-stable.desktop
$ rm -fr ~/.cache/vivaldi
$ rm -fr ~/.config/vivaldi  (optional)
```

### <a id="wikis">See also, wikis at Arch Linux

* [Chromium](https://wiki.archlinux.org/title/Chromium)
* [Google Chrome](https://wiki.archlinux.org/title/Google_chrome)
* [Hardware video acceleration](https://wiki.archlinux.org/title/Hardware_video_acceleration)
* [Vivaldi](https://wiki.archlinux.org/title/Vivaldi)

