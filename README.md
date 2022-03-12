# RPiMotionCam

https://user-images.githubusercontent.com/44409029/156776492-51b29ff9-b3c7-4e85-9587-974a2d363a77.mp4

## Raspberry Pi motion surveillance camera with email notification and gdrive storage
[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg)](https://opensource.org/licenses/BSD-3-Clause)<br/><br/>

This application is a fully-featured security Raspberry Pi camera. It can be built with the Raspberry Pi 4, 3 or the tiny Zero 2.<br/>
With an inexpensive RPi V1 camera ($ 7,=), you'll have your security camera up and running in no time.<br/>
You will automatically receive an email when the camera detects a movement.<br/>
At the same time, a optional video recording will be saved to SD-card, USB stick or your Google drive.<br/>
You can view your footage in your browser at any time.<br/>
You don't need to be able to program.<br/> However, the used C++ source code comes with the image.<br/>

------------

## Installation.

- Get a SD-card (min 16 GB) which will hold the image. 
- For the Raspberry Pi **4** download the image Motion_RPi4.xz (2.16 GByte!) from our [Gdrive](https://drive.google.com/file/d/1OrAM2Ls7Mwfz1Iyn6cYq6VS0UUDWQxWL/view?usp=sharing) site.
- For the Raspberry Pi **3** download the image Motion_RPi3.xz (2.16 GByte!) from our [Gdrive]() site.
- For the Raspberry Pi **Zero 2** download the image Motion_RPiZ2.xz (2.16 GByte!) from our [Gdrive]() site.
- Flash the image on the SD-card with the [Imager](https://www.raspberrypi.org/software/) or [balenaEtcher](https://www.balena.io/etcher/).
- Insert the SD-card in your Raspberry Pi.
- Wait a few minutes, while the image will expand to the full size of your SD card.
- No WiFi installed. Password: ***motion***

------------

## Preparations.
There are a few settings needed before the application will work properly.<br/>
- First, of course, you need an internet connection. Setup your WiFi or Ethernet as usual.<br/>
After reboot, you must have video footage in your browser. Just give the Raspberry Pi IP, like the http://192.168.178.32 used in the demo video.
- If you want to receive emails and/or store recordings at Google drive, you will need an Google account. Since all your personal login information can be found in the Raspberry Pi, we recommend a separate Google account for this application. Just for safety reasons. 
- Use `$ sudo nano /etc/msmtprc` to give the Google login information. See this [WiKi page](https://github.com/Qengineering/RPiMotionCam/wiki/Email-notification).
- Allow Gmail to process less secure applications. See the [WiKi page](https://github.com/Qengineering/RPiMotionCam/wiki/Email-notification). 
- Get the authorization key from Google for gdrive. Give `$ gdrive about`. See the [WiKi page](https://github.com/Qengineering/RPiMotionCam/wiki/Gdrive-installation#authorization-key). You don't have to install gdrive, it's already on board. You only need the key.
- The following action is the settings file. Apart from the threshold, you must provide the internet addresses. See the [WiKi page](https://github.com/Qengineering/RPiMotionCam/wiki/Settings).
- ***Most important, set the overlay*** active. SD cards wear out when written and can cause your system to crash. Read this [WiKi page](https://github.com/Qengineering/RPiMotionCam/wiki) carefully to see which solution is best for you.

------------

## Donations.
Writing the app was a lot of work. Still, we want to give you all the source code for free. As well as the FFmpeg streaming solutions.<br/>
However, we really appreciate it if you show your appreciation by making a donation.<br/>
[![paypal](https://qengineering.eu/images/TipJarSmall4.png)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=CPZTM5BB3FCYL)<br/><br/><br/>

![output image]( https://qengineering.eu/images/GdriveMotion.webp)

------------

## How does it work?
The application detects movements in a scene and can trigger a motion event. This motion event can send an email and/or start a recording.<br/><br/>
A common background image is generated from an average of many previous video frames. The latest frame is subtracted from this background. Only pixels values not equal to the corresponding background pixel value are marked. All marked pixels are counted, and a percentage is calculated. This percentage can trigger the motion event if it's greater than the `set_trigger` setting.<br/>
You will need to experiment with `set_trigger` and `reset_trigger` to see what suits you best in your situation. Detecting a person in front of your door requires other settings than detecting the neighbours' car in the backyard.<br/><br/> 
At the same time, this frame is averaged with the other recently captured images. It means that once a movement is stopped, it gradually vanish into the background. Like the car in the video above. When it was parked, it slowly disappears. The time it takes to vanish is defined by the constant `TAU_BACKGROUND` at line 36 in General.h. If you want to change it, you need to re-compile the MainEvent app. It is not a setting like the ones in Settings.txt.<br/><br/> 
Note that sudden changes in light can also trigger an event. Think of clouds sliding in front of the sun or swaying branches of a tree. There is not much you can do about it. In some cases, it helps by excluding certain regions in the frames from the background subtractor. For instance, a busy street with much traffic at the upper part of your footage. You have to program it yourself in the MainEvent app with OpenCV.<br/>

------------

## Motivation.
The main goal was to write the simplest possible app without compromising functionality. We think we have succeeded in that. Many functions are controlled with just a few small C++ programs. An experienced programmer can understand our programs within half a day instead of spending days struggling to encapsulate the functionality in a complex framework like MotionOS.<br/><br/>
Of course, this strategy comes with a price.<br/>
- We use the Debian 10 (Buster) 32-bit operating system. FFmpeg is the only streaming framework that frees the CPU complete from all streaming activity. However, the current FFmpeg does not support the new camera functionality found on the Bullseye release for the Raspberry Pi. Once FFmpeg handles the libcamera completely via the GPU, we can transfer the app to Bullseye. Until then, we'll have to stick with Buster. (Which, by the way, is not a severe punishment).
- If we can make something work with one or two lines of script, we use it with the `system()` call, instead of writing a lot of C++ code to emulate the same functionality.
- We don't check anything in advance. No extensive check of disk space, USB mounting, email verification, etc. 

------------

## C++ code.
The C++ code is available and allows you to modify the application to your needs. All resources are in the `software` folder. As well as the Code::Blocks project files to build the apps. For information on how to run a project, see our guide to [OpenCV and Code::Blocks](https://qengineering.eu/opencv-c-examples-on-raspberry-pi.html).<br/><br/>
:point_right: ***Before you start programming, make sure you have removed the overlay functionality, if enabled.*** <br/><br/>
The autostart links to executable files in `/usr/local/bin`. You need to move your executables from your project file `/bin/Release/` to `/usr/local/bin` once programming is complete. Otherwise, autostart will not use your last exe.<br/>
More information about programs found at `/usr/local/bin` on this [Wiki page](https://github.com/Qengineering/RPiMotionCam/wiki/Usr-Local-Bin).

------------

## Good to know.

You have a latency of about 10 seconds. This time is inherent to the HLS streaming. It takes some time to collect all the information from the stream, get the individual packets and 'glue' them together into one video stream.<br/>
By the way, thanks to this latency, you will receive your emails 5 seconds before the actual movement is visible in your browser so you can log in.<br/><br/>
Many free sites convert an email to a text message. At the same time, you can port forward your Raspberry Pi, making it accessible with your private IP and user-defined port number. It is even possible to get a nice domain name for your camera (https://wwww.MyBackYardRPi.com). Google something like "free DNS for your IP camera".<br/>

------------

## Tip.

We used the cheap RPi camera V1 for € 6,70. It works fine. However, the tiny plug from the embedded sensor to the PCB often can be loose. Somehow the software still supported the camera but didn't receive any video anymore. It took quite a while before we discovered the cause; the connector. Once glued, it now functions perfectly.<br/><br/>
![output image]( https://qengineering.eu/images/CheapRPiCam.webp)<br/><br/>
You can use the command `$ vcgencmd get_camera` to see if your camera is working.<br/><br/>
![output image]( https://qengineering.eu/images/CamDetect.webp)<br/>

