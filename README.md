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
Terminal=false
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
**If you use our image it's already installed.**<br/>
However, if you want to install it on another RPi, here are the steps.<br/>
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
$ go get github.com/prasmussen/gdrive
$ cd ~/go/bin
$ sudo cp ./gdrive /usr/local/bin
```
The next step is getting the authorization key from Google. With the key in place, you can access your Gdrive.<br/>
The key is only valid for the current Raspberry Pi and is located at `/home/pi/.gdrive` <br/>
Note that everyone with access to this file has access to your Gdrive. Be careful.<br/><br/>
Give the command `$ gdrive about` and copy-paste the URL in your browser.<br/>
![output image]( https://qengineering.eu/images/Gdrive_About.webp )<br/><br/>
After logging into your Google account, you will be asked if you want to give your project access to the gdrive.<br/>
![output image]( https://qengineering.eu/images/Google_Accounts_Gdrive.webp )<br/><br/>
Once allowed, you get a unique code that can be copy-pasted back to the terminal screen.<br/>
![output image]( https://qengineering.eu/images/Gdrive_Key.webp )<br/><br/>
Now your gdrive is up and running. You can test it with the `$ gdrive list` command.
## email notification
**If you use our image it's already installed.**<br/>
We use gmail as it can forward your mails to any other address.<br/>
However, if you want to install the email engine on another RPi, here are the steps.<br/>
```
$ sudo apt-get install msmtp ca-certificates
$ sudo nano /etc/msmtprc
```
![output image]( https://qengineering.eu/images/Email_google.webp )<br/><br/>
Before you can send an email to any address, you'll need to enable Gmail to process your emails from the Raspberry Pi, a device that doesn't meet all of Google's security requirements. To do so, log in to your profile on your google account. Allow access to less secure apps here.<br/><br/>
![output image]( https://qengineering.eu/images/LessSecureApp2.webp )<br/><br/>
Now you can test the connection with the next command:<br/>
`echo -e "Subject: Test Mail\r\n\r\nThis is a test mail" |msmtp --from=default -t johndoe123@his_mail.us`<br/>
Of course, you have to replace _johndoe123@his_mail.us_ with your own email address.<br/>
## Applications architecture
The motion camera consists of four parts.<br/> All four run in a separate process, having their unique PID. We use processes instead of threads to make sure there isn't a time-consuming thread holding an image stream. The last thing you want is for the app to respond to long-ago events because of an overflow in an image buffer.<br>
1) The first part is the FFmpeg streamer and the Nginx server. The FFmpeg wraps the h264 video stream to an RTMP friendly flv format. The Nginx with the RTMP add-on converts the video to HLS, a modern browser format. You can watch the video in any browser. The FFmpeg software only uses the GPU, a great advantage when you later need all the CPU power available for deep learning models. The Nginx server also runs very quietly in the background, consuming a max of 1% CPU.
2) With inference times up to 0.5 sec, timing is crucial. You cannot have any buffering in the chain. Every image processed needs to be with as little latency as possible. Here, the Qstreamer comes in place. It constants grab frames from the HLS stream FFmpeg and Hginx generates. Once an image is processed, a flag is set, indicating a request for a new frame. The Qstreamer will respond to the request by sending the latest frame just received. 
3) MotionEvent analyses the images. Once a movement is detected it send the appropiate signals to other parts of the application.
4) The last part is the recording. If a USB memory device is connected, the recorder will use it. If not, they are stored at `var/www/html/rec`. If Gdrive is enabled, the recording uploads upon completion. The original recording is deleted if this option is set. Be careful when using the overlay feature (the Raspberry Pi's read-only safety mode). All writes are in RAM. Make sure you have enough space. The program does not check this. Another FFmpeg stream takes care of the recording. This way, you also save the images 5 seconds before the movement event. In other words, you see the postman walking up the porch instead of just walking out of your yard just because of a time-lapse when the shooting starts. The latter will be the case if you use Nginx's recording facilities.
## Gparted
## Settings
