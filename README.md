# setup-logitechc920-webcams-server
Simple guide to setting up multiple Logitech c920 webcams as a server on either a raspberry pi or linux debian box

Major shout out to Redonkuless on discord for his suberb assistance helping me set this up.

###  NOTE: All IP addresses used are currently 10.0.0.11.  These need to be modified to match the IP address of the web cam server; e.g. a Raspbian Bullseye distribution OR similar Debian distro.

## Commands:

```apt install ffmpeg```

```apt install nginx libnginx-mod-rtmp```

```apt install libnginx-mod-rtmp```

## Move nginx.conf to NUC
Copy nginx.conf as root to /etc/nginx

```service nginx restart```

## Insert this into OBS as media source with NUC IP address:
```rtmp://10.0.0.11:1935/live/mystream```

## Insert this into OBS as media source with NUC IP address:
```rtmp://10.0.0.11:1935/live/mystream2```


## EITHER:
// Change IP address to match NUC instead of pi
```
ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video0 -c:v libx264 -c:a libmp3lame -f flv rtmp://10.0.0.11:1935/live/mystream
```
```
ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video2 -c:v libx264 -c:a libmp3lame -f flv rtmp://10.0.0.11:1935/live/mystream2
```
## OR for automation:
Create startupwebcam.sh with an editor in home directory and place the following code in there; not change IP address to NUC IP:
```
#!/bin/bash

while true; do
    if systemctl is-active --quiet nginx; then
        break
    fi
    sleep 5
done


# Wait for video devices to become available
while [ ! -e /dev/video0 ] && [ ! -e /dev/video2 ]; do
    sleep 5
done

sleep 40

/usr/bin/ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video0 -c:v libx264 -c:a libmp3lame -f flv rtmp://10.0.0.11:1935/live/mystream &
/usr/bin/ffmpeg -thread_queue_size 512 -ar 44100 -acodec pcm_s16le -f s16le -ac 2 -channel_layout 2.1 -i /dev/null -thread_queue_size 512 -f v4l2 -pix_fmt yuv420p -codec:v mjpeg -framerate 25 -video_size 800x600 -i /dev/video2 -c:v libx264 -c:a libmp3lame -f flv rtmp://10.0.0.11:1935/live/mystream2 &
```
Then send the command:

```chmod +x startupwebcam.sh```

// Add these lines before exit 0 in /etc/rc.local using your favourite editor:
```
# uncomment this line for debugging:

#/home/pi/startwebcam.sh > /home/pi/startwebcam.txt 2>&1

# uncomment this line for production:

/home/pi/startwebcam.sh > /dev/null 2>&1
```
