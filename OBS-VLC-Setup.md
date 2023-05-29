# OBS Studio setup

## Note change IP addresses to server source; Raspbian Bullseye or Debian distro.

Add a media source to OBS Studio.

Untick local file as source.

Enter this as input:

```
rtmp://10.0.0.11:1935/live/mystream
```

Repeat with second webcam:

```
rtmp://10.0.0.11:1935/live/mystream2
```

# VLC Media Player setup

In Media | Open Network Stream enter the following text:

```
rtmp://10.0.0.11:1935/live/mystream
```

or 

```
rtmp://10.0.0.11:1935/live/mystream2
```


