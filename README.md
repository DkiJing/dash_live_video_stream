# dash_live_video_stream
## Introduction
This project is based on nginx server to implement a live video application.
## Prerequisite
+ nginx-rtmp-module
+ ffmpeg
+ dash.js
## Setup nginx-rtmp-module
Please go to the [official site](https://github.com/arut/nginx-rtmp-module/wiki/Installing-via-Build).
## Configure of nginx-rtmp-module
```
# RTMP config
rtmp {
    server {
        listen 1935; # Listen on standard RTMP port
        chunk_size 4096;
        application live{
          live on;
          # push different streams to nginx server by ffmpeg. $app is live and $name is zhou in this example.
           exec /usr/local/bin/ffmpeg -i rtmp://localhost/$app/$name
           -c:v libx264 -c:a aac -s 1024x576 -f flv rtmp://localhost/play/$name_hi
           -c:v libx264 -c:a aac -s 640x360 -f flv rtmp://localhost/play/$name_med
           -c:v libx264 -c:a aac -s 320x180 -f flv rtmp://localhost/play/$name_low;
        }
        
        application play {
            live on;
            record all;
            record_path /Users/joycez/杂类文件/videos;
            dash on;
            dash_nested on;
            dash_repetition on;
            dash_path /usr/local/var/www/html/dash;
            dash_fragment 4;
            dash_playlist_length 120;
            dash_cleanup on;
            # devide video stream into three different streams.
            dash_variant _low bandwidth="256000" width="320" height="180";
            dash_variant _med bandwidth="832000" width="640" height="360";
            dash_variant _hi bandwidth="2048000" width="1024" height="576" max;
                
        }
    }
}
# End 
```
## Setup dash.js
Please go to the [official site](https://github.com/Dash-Industry-Forum/dash.js).
## Configure of HTTP server
```
# HTTP config
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen 8080;
        server_name  localhost;

       location /stat{
           rtmp_stat all;
           rtmp_stat_stylesheet stat.xsl;
       }
       location /stat.xsl{
           root /usr/local/Cellar/nginx-rtmp-module-dev;
       }
        location /dash{
            root /usr/local/var/www/html;
            add_header Cache-Control no-cache;
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
            add_header 'Access-Control-Allow-Headers' 'Range';
        }
        # Put the html file into the root path.
        location /DPlayer-master{
              root /usr/local/var/www/html;
            }
        location /{
            root /usr/local/var/www/html;
        }
        location /dash-js{
        root /usr/local/var/www/html;
        }
    }
    include servers/*;
}
# End
```
## Video playing page
I use [DPlayer](https://github.com/MoePlayer/DPlayer) to implement switching different version of resolution.
```
<script>
    const player = new DPlayer({
        container: document.getElementById('dplayer'),
        autoplay: true,
        video:{
            quality:[
            {
                name: 'high',url: 'http://localhost:8080/dash/zhou_hi/index.mpd',type: 'dash'
            },
            {
                name: 'median', url: 'http://localhost:8080/dash/zhou_med/index.mpd',type: 'dash'
            },
            {
                name: 'low',url: 'http://localhost:8080/dash/zhou_low/index.mpd',type: 'dash'
            },
            {
                name: 'auto', url:'http://localhost:8080/dash/zhou.mpd',type:'dash'
            }],
            defaultQuality: 0
        }
    }); 
</script>
```
## Operation flow
Start nginx server `sudo /usr/local/nginx/sbin/nginx`

Close nginx server `sudo /usr/local/nginx/sbin/nginx -s stop`

Push video stream by OBS

>Streaming Service: Custom

>Server: rtmp://localhost/live

>Stream Key: test
