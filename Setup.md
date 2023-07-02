###  NOTE: All IP addresses used are currently 10.0.0.11.  These need to be modified to match the IP address of the web cam server; e.g. a Raspbian Bullseye distribution OR similar Debian distro.  You may need to remove and reinstall certain applications that may be broken by performing all of these procedures since the sources.list file is using a different tree to source the packages for your linux distro. 

## Edit your /etc/apt/sources.list as follows:

```
#deb http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi

deb http://ftp.uk.debian.org/debian bullseye main

# Uncomment line below then 'apt-get update' to enable 'apt-get source'

#deb-src http://raspbian.raspberrypi.org/raspbian/ bullseye main contrib non-free rpi
```

## From the command line run the following:

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 648ACFD622F3D138

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 605C66F00D6C9793

sudo apt update

apt install ffmpeg

apt install nginx libnginx-mod-rtmp

apt install libnginx-mod-rtmp
```

## Move nginx.conf to NUC
Copy nginx.conf as root to /etc/nginx

```service nginx restart```

## Insert this into OBS as media source with NUC IP address:
```rtmp://10.0.0.17:1935/live/mystream```

## Insert this into OBS as media source with NUC IP address:
```rtmp://10.0.0.17:1935/live/mystream2```


## EITHER:
// Change IP address to match NUC instead of pi
```
ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video0 -c:v libx264 -preset ultrafast -tune zerolatency -c:a libmp3lame -f flv rtmp://10.0.0.17:1935/live/mystream &
```
```
ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video2 -c:v libx264 -preset ultrafast -tune zerolatency -c:a libmp3lame -f flv rtmp://10.0.0.17:1935/live/mystream2 &
```
```
ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video4 -c:v libx264 -preset ultrafast -tune zerolatency -c:a libmp3lame -f flv rtmp://10.0.0.17:1935/live/mystream3 &
```
## OR for automation:
Create startupwebcam.sh with an editor in home directory and place the following code in there; not change IP address to NUC IP:
```
#!/bin/bash

# Wait for nginx service to become active
while ! systemctl is-active --quiet nginx; do

sleep 5

done

# Wait for video devices to become available
while [ ! -e /dev/video0 ] && [ ! -e /dev/video2 ] && [ ! -e /dev/video4 ]; do

sleep 5

done

sleep 40

# Start ffmpeg processes with optimized settings
ffmpeg -thread_queue_size 64 -f v4l2 -input_format mjpeg -i /dev/video0 -c:v libx264 -preset ultrafast -tune zerolatency -c:a aac -f flv rtmp://10.0.0.17:1935/live/mystream &
ffmpeg -thread_queue_size 64 -f v4l2 -input_format mjpeg -i /dev/video2 -c:v libx264 -preset ultrafast -tune zerolatency -c:a aac -f flv rtmp://10.0.0.17:1935/live/mystream2 &
ffmpeg -thread_queue_size 64 -f v4l2 -input_format mjpeg -i /dev/video4 -c:v libx264 -preset ultrafast -tune zerolatency -c:a aac -f flv rtmp://10.0.0.17:1935/live/mystream3 &
```
Then send the command:

```chmod +x startupwebcam.sh```

// Add these lines before exit 0 in /etc/rc.local using your favourite editor:
```
# uncomment this line for debugging:

#/home/pi/startupwebcam.sh > /home/pi/startwebcam.txt 2>&1

# uncomment this line for production:

/home/pi/startupwebcam.sh > /dev/null 2>&1
```
