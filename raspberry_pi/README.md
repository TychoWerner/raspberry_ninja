
<img src="https://user-images.githubusercontent.com/2575698/127804804-7ee38ebd-6f98-4242-af34-ac9ef0d9a82e.png" width="400">

## Installation on a Raspberry Pi

It is recommended to use the provided Raspberry Pi image, as the install process is otherwise quite challenging.  If using an image, you will want to update the code afterwards to ensure you're running the newest version.

#### Installing from the provided image

Download and extract the image file:
https://drive.google.com/file/d/1f1PCEpKBLT3KVZSRR6pYwywt9TKjPmwW/view?usp=sharing

(** note, Someone reported that their macOS system could not unzip it, but worked fine on Windows)

Write the image to a SD / microSD card; at least 8GB in size is needed, using the appropriate tool. 

On Windows, you can use Win32DiskImager (https://sourceforge.net/projects/win32diskimager/) to write this image to a disk.  If you need to format your SD card first, you can use the SD Card Formatter (https://www.sdcard.org/downloads/formatter/).  

(balenaEtcher also works for writing the image, but using the official Raspberry Pi image writer may have problems.)

#### Setting up and connecting

Before loading the uSD card into your RPi, you can configure some basic settings on the drive itself via Windows/Mac/Linux.

You can setup the WiFi by creating a copy of `X:/boot/wpa_supplicant.conf.sample` named `wpa_supplicant.conf`.  Editing that file, change the ssid and psk lines with your WiFi's SSID and password.  This will ensure it auto connects to your WiFi on first boot.

You can also open `X:/boot/config.txt` in notepad to uncomment a line to enable support for the HDMI to CSI adapter.  There's a file also called `setup_c779.sh` in the raspberry_pi folder that also needs to be run once booted in, if using the C779 HDMI to CSI adapter at least. Probably not needed with the B10X boards, but can't confirm.  You will want to run it before connecting any input to the HDMI input port.

To connect, use a display and keyboard, or you can SSH into it as SSH on port 22 if enabled.  If you didn't configure the WiFi, you connect either via USB or Ethernet instead. Refer to the Raspberry Pi documentation for more help on that topic.

Note: By default the provided image will have SSH enabled, so if security is a concern, disable SSH. It is also suggested that you change the username and password to something more secure. I woudld also suggest not installing any BTC wallets on this image or anything like that, as security can't be guaranteed.

#### Login info

Login information for the device is:
```
username: pi
password: raspberry
```

You can then run `sudo raspi-config` from the command-line to configure the Pi as needed. If using the Raspberry Pi camera or another CSI-based camera, you'll want to make sure it's enabled there if using any CSI-based device.

You will probably want to also update the pi with `sudo apt-get update && sudo apt-get upgrade`, to snure it's up to date.  You can also run `sudo raspi-config` and update the RPi that way, along with updating the bootloader if desired (on a pi4 at least).

#### Installing from scratch

If you do not want to use the provided image, you can try to install from scratch, but be prepared to lose a weekend on it. Please see the install script provided, but others exist online that might be better. Gstreamer 1.14 can be made to work with VDO.Ninja, but GStreamer 1.16 or newer is generally recommend; emphasis on the newer.

The official image setup for the Raspberry Pi is here: https://www.raspberrypi.org/documentation/installation/installing-images/windows.md

The `installer.sh` file in this folder contains the general idea on how to install an updated version of Gstreamer on a Raspberry Pi. Expect to waste a week of time on it, chasing compiling issues I'm sure.

## Running things

Once connected to you Pi, you can pull the most recent files from this Github repo to the disk using:

```
sudo rm raspberry_ninja -r
git clone https://github.com/steveseguin/raspberry_ninja.git
cd raspberry_ninja
python3 publish.py --streamid YOURSTREAMIDHERE --bitrate 4000
```

After cloning the code repository, if you have any problems or wish to update to the newest code in the future, run `git pull` from your raspberry_ninja folder. This should download the most recent code. You will need to clear or stash any changes before pulling though; `git reset --hard` will undo past changes. `git stash` is a method to store past changes; see Google on more info there though.

### Camera considerations

If using an **original Raspberry Pi Zero**, an official Raspberry Pi CSI camera is probably the best bet, as using a USB-based camera will likely not perform that well. USB works pretty okay on a Raspberry Pi 4, as it has enough excess CPU cores to handle decoding motion jpeg, audio encoding, and the network overhead, but the Pi Zero original does not.

On a Pi Zero W original, 640x360p30 works fairly well via a USB camera. At 1080p though, frame rates drop down to around 5 to 10-fps though (10-fps with omx, but glitches a bit).  With an official CSI camera, 1080p30 is possible on a Pi Zero W original, but you might get the occassional frame hiccup still.

Some USB devices may struggle with audio/video syncronization. Video normally is low latency, but audio can sometimes drift a second or two behind the video with USB audio devices. We're trying to understand what causes this issue still; dropping the video resolution/frame rate can sometimes help though. It also doesn't seem to manifest itself when using Firefox as the viewer; just Chromium-based browsers.

If you are using a Raspberry Pi 4, then you should be pretty good to go at this point, even at 1080p30 MJPEG over USB 2.0 seems to work well there. You might contend with audio/video sycronization issues if using a USB camera/audio source still, but hopefully that issue can be resolved shortly. 

If you are using a Raspberry Pi 2 or 3, you might want to limit the resolution to 720p, at least if using a USB camera source.

If using the CSI camera, the hardware encoder often works quite well, although it might still be best to limit the resolution to 720p30 or 360p30 if using an older raspberry pi zero w. The Raspberry Pi Zero 2 however works quite well at 1080p30 with the official Raspberry Pi cameras.

To enable the CSI camera, you'll need to add `--rpicam` to the command-line, as the default is USB MPJEG.  You may need to run `sudo raspi-config` ane enable the CSI camera inteface before the script will be able to use it. Some CSI cameras must be run with `--v4l2` instead, and some others require custom drivers to be installed first. Sticking with the official raspberry pi cameras is your best bet, but the $40 HDMI-to-CSI adapter and some knock off Raspberry Pi CSI cameras often will work pretty well too.

To enable RAW-mode (YUY2) via a USB Camera, instead of MJPEG, you'll need to add `--raw` to the command line, and probably limit the resolution to around 480p. 

If using an HDMI adapter as a camera source, you may need to adjust things at a code level to account for the color-profiles of your camera source. `--bt601` is an option for one stanard profile, with the default profile set at `coloimetry=2:4:5:4` for a Raspberry Pi. 10-bit and interlaced video probably isn't a good idea to attempt to use, and if possible setting the HDMI output of your camera to 8-bit 1080p24,25 or 30 is the highest you should probably go.

###### Please return to the parent folder for more details on how to run and configure

## Setting up auto-boot

There is a service file included in the raspberry_pi folder that sets the raspberry pi up to auto-boot.  You will need to modify it a tiny bit with the settings to launch by default, such as specifying the stream ID and camera settings, such as --rpicam or --rpi or --hdmi, etc.

To edit the file, you can use VIM.
```
cd ~
cd raspberry_ninja
cd raspberry_pi
sudo vi raspininja.service
```
To use VIM, press `i` to enter text edit mode.  `:wq` will let you save and exit.

Once done editing the file, you can set it up to auto launch on load and to start running immediately.
```
sudo cp raspininja.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable raspininja
sudo systemctl restart raspininja
```
To check if there were any errors, you can run the status command:
```
sudo systemctl status raspininja
```

Things should now auto-boot on system boot, and restart if things crash.  


## Details about cameras and performance

Using a CSI-based camera with a Raspberry Pi will currently give better results than a USB-based one; or at least. The official Raspberry Pi cameras can work with `--rpicam`, and the results are quite good. Some other knock off cameras can still be hardware encoded, but may need to be specified with `--v4l2`, which doesn't work quite as well as the official source element. Still, the results are often good up to 1080p30.

There are some non-supported cameras that use the CSI port, like the Arducam Sony IMX327 sensor-based cameras, as those may not have any proper driver support added. You can get those to work if they offer Gstreamer-based drivers though, however installing them may prove quite challenging. I'll provide pre-built images that support such devices when I have them working.

Lastly, unless using the RPi Compute Module, any HDMI to CSI adapter for the RPi will be limited to 25-fps.  With a 4-lane camera and the compute module, you might be able to do 1080p30 with HDMI to SCI adapters.  HDMI to CSI adapters do not include audio, unless you route the audio from the adapter to the I2S pins, which may require some tinkering to setup.


## Optimizing the Pi to reduce packet loss

Before: ![image](https://user-images.githubusercontent.com/2575698/146271521-ed9f8742-d584-4214-938c-687388b658bf.png)

After: ![image](https://user-images.githubusercontent.com/2575698/146271560-852984cb-6cd0-47d2-a03d-c2e78358652a.png)

The Raspberry Pi doesn't have the greatest WiFi adapter, and even the Ethernet has some issues, so it needs some added love to keep up.  By applying some optimizations to the Raspbian OS, you can increase stability, resulting in less packet loss, and in turn, a more stable frame rate. (Some of these changes are already applied to the provided Raspberry Pi image (v3), but they will need to be manually applied if building from scratch.)

The following changes resulted in a rather sharp improvement in frame rate stability; if you find more tweaks, please submit them! :D

Okay, the first optimization was with my pi4 that has active cooling; I locked the CPU cores into performance mode with the force_turbo flag.
```
force_turbo=1
```
To do that, I added the above to `/boot/config.txt`.

You'll might need to adjust your arm_freq to match what your device can handle also, depending on model and cooling. A stable CPU seems to help.

Next,  added the following to this file:  `/boot/cmdline.txt`
```
isolcpus=0
```
This removes core 0 from user access, and allows it to be dedicated as a core for the wifi stack. Or so that's my understanding.  This won't work with a single-core raspberry Pi I guess, and may hinder h264-software based encoding options, but it did make an improvement (x264 vs rpicam)

I also added a bunch of buffers to the gstreamer-python code; some did more harm then good, but overall no great impact.  I left them in to just make me feel better, but you can set limits on them if there are problems. These should be updated on Github now, so be sure to pull the most recent code.

Also, if using Ethernet, to avoid packet loss on the Pi4 when connected to gigabit, run the following: 
```
sudo ethtool -s eth0 speed 100 duplex full autoneg off
```
You can make this change apply no boot by editing `/etc/rc.local` and adding `ethtool --change enp1s0 speed 100 duplex full autoneg off` before the `exit 0` line.

This reduces the speed to 100mbps, instead of gigabit, but it also can dramatically reduce packet loss on a Raspberry Pi. 100mbps is more than enough anyways.
You may want to have the command auto-run on boot, just to be safe, but if you're not using Ethernet, it may not be needed. 

You can further optimize the system by disabling Bluetooth and disabling network sleep mode.  For details on what you can try there, see this guide: 
https://forums.raspberrypi.com/viewtopic.php?t=138312&start=50#p1094659   You might want to keep audio on and other deviations from those instructions, if bothering to follow it at all.

Finally, just to add some buffering onto the Viewer side as well, in Chrome (edge/electron), I added &buffer=300 to the view link.  The VDO.Ninja player has like 40 to 70ms already added, but increasing it to 300 or so will help pre-empt any jitter delays, avoiding sudden frame loss. This is not required, but seems to help a bit at the cost of added latency.  You may want to increase it upwards of 1000ms to see if it helps further, or go without any added buffer entirely.

If using the buffer option, the view link might look like this `https://vdo.ninja/?password=false&view=9324925&buffer=300`

The buffer command is compatible with OBS v27.2 on PC and newer, but not earlier versions on PC.

### Firmware

Updating the firmware for the Raspberry Pi 4 might help with some USB controller or networking issues. 
```
# check if newest version is needed
sudo rpi-eeprom-update

# If not up to date, then we can update
sudo raspi-config
# Advanced Options -> Bootloader Version -> Latest
# Reboot when prompted
```
You can also try updating the firmware/system for other RPI boards using
```
sudo apt update
sudo apt full-upgrade
sudo reboot
```
### Problem with Raspberry Pi Camera and Pi Zero 2

I ran into an issue where the RPI Camera (v1.3 and v2.x) were not working on my Raspberry Pi Zero 2.

Typically you'd check `sudo raspi-config` and make sure the camera is enabled via the interface options.  If that doens't work, you'd then check to make sure the cable on the camera board is not loose.

None of those worked, but adding the following to the `/boot/config.txt` and rebooting file fixed things.
```
start_file=start_x.elf
fixup_file=fixup_x.dat
```
