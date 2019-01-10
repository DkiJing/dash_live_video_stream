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
          exec /usr/bin/ffmpeg -i rtmp://120.78.166.34:1935/$app/$name
           -c:v libx264 -c:a aac -strict -2 -s 1024x576 -f flv rtmp://120.78.166.34/play/$name_hi
           -c:v libx264 -c:a aac -strict -2 -s 640x360 -f flv rtmp://120.78.166.34/play/$name_med
           -c:v libx264 -c:a aac -strict -2 -s 160x90 -f flv rtmp://120.78.166.34/play/$name_low;
        }

        application play {
            live on;
            dash on;
            dash_nested on;
            dash_repetition on;
            dash_path /var/tmp/dash;
            dash_fragment 4;
            dash_playlist_length 120;
            dash_cleanup on;
            dash_variant _low bandwidth="128000" width="160" height="90";
            dash_variant _med bandwidth="416000" width="640" height="360";
            dash_variant _hi bandwidth="1024000" width="1024" height="576" max;
                
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
    server {
        listen 8080;
	server_name 120.78.166.34;

        location /dash{
            root /var/tmp;
            add_header Cache-Control no-cache;
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
            add_header 'Access-Control-Allow-Headers' 'Range'; 
	}

        location /dash.js{
            root /var/www;
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
Server: rtmp://120.78.166.34/live
Stream Key: zhou

Pull video stream by browser
`http://120.78.116.34:8080/baseline.html`
