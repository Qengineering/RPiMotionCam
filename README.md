# RPiMotionCam



https://user-images.githubusercontent.com/44409029/156776492-51b29ff9-b3c7-4e85-9587-974a2d363a77.mp4



## Raspberry Pi motion surveillance camera with email notification and gdrive storage



https://drive.google.com/file/d/1vH6puq3pB41yC-Q6nKeM244vz2YSaZDE/view?usp=sharing

## Autostart
The autostart is defined in `/etc/xdg/lxsession/LXDE-pi/autostart`<br/>
Once opened you see:
```
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
@xscreensaver -no-splash
@lxterminal -e /usr/local/bin/StartCam.sh
```
The last line starts the motion camera application. You may remove this line.<br/>
Do not alter the others because the RPi desktop needs them to start properly.<br/>
```
#!/bin/sh
sleep 10
MotionEvent
```
The simple script waits for 10 seconds to give the RPi time to establish the internet and mail connection.<br/>
After this time, it starts MotionEvent, also located in `/usr/local/bin`.

## Desktop icon
The desktop icon is defined in `~/Desktop/Camera.desktop`<br/>
```
[Desktop Entry]
Version=1.1
Type=Application
Encoding=UTF-8
Name=Motion camera
Comment=Motion camera
Icon=/home/pi/software/Motion/CamIcon.png
Exec=lxterminal -e MotionEvent
Name[en_GB]=Motion
```
## Menu icon
The menu entry is defined in `/usr/share/applications/motion-cam.desktop`<br/>
```
[Desktop Entry]
Name=Motion Camera
GenericName=Motion Camera
Comment=RPi motion camera
Exec=lxterminal -e /usr/local/bin/MotionEvent
Icon=/home/pi/software/Motion/CamIcon.png
Categories=AudioVideo;
Terminal=true
Type=Application
```
## Changing the resolution
You can change the resolution of the recorded video, depending on the supported formats of the RPi camera and your computing power.<br/>
For instance, a Raspberry Pi Zero will have troubles with a large format, as where the RPi 4 may still run quietly.<br/>
The format for the background subtractor will always be 320x240. The output is a percentage, no need for CPU absorbing large resolutions.<br/>
See line 43 in Qstreamer.cpp
```
cv::resize(frame, norm_frame, cv::Size(320, 240));
```
The resolution is defined at the end of `/etc/rc.local`<br/>
In the example below 680x480.
```
#start streaming (notice the & at the end)
ffmpeg -input_format h264 -f video4linux2 -video_size 680x480  -framerate 15 -i /dev/video0 -c:v copy -an -f flv rtmp://localhost/live/rpi &

exit 0
```
You can get the available formats with the next command `$v4l2-ctl --device=/dev/video0 -D --list-formats-ext`
## Gdrive installation
Gdrive is a command-line tool for managing your Google Gdrive from the Raspberry Pi.<br/>
Start with the installation of Googles GO.<br/>
Find `go1.17.7.linux-armv6l.tar.gz` at the [download page](https://go.dev/dl/).<br/>
You have probably an armv8 (RPi Z2, 3 or 4), but the armv6 will work and is the only option here.<br/>
```
$ wget https://go.dev/dl/go1.17.7.linux-armv6l.tar.gz
$ sudo tar -C /usr/local -xzf go1.17.7.linux-armv6l.tar.gz
$ export PATH=$PATH:/usr/local/go/bin
# check the version
$ go version
# output go version go1.17.7 linux/arm
```
With GO working, you can install gdrive.
Because there is no Python wheel, we need to install it from the source. Hence the need for GO.<br/>
```
go get github.com/prasmussen/gdrive
```
The next step is getting the authorization key from Google. With the key in place, you can access your Gdrive.<br/>
The key is only valid for the current Raspberry Pi and is located at `/home/pi/.gdrive` <br/>
Note that everyone with access to this file has access to your Gdrive. Be careful.<br/>
