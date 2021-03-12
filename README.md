# Jellyfin-Pyload-with-Docker-and-SSL-Let-s-encrypt
> Install Jellyfin with a direct download manager and torrent + SSL
> Jellyfin 10.7.0 + PYLoad ( docker ) + Deluge ( docker ) + Nginx for reverse proxy

Ubuntu Installation (18.04 ++)

## Installation Jellyfin, PyLoad and Deluge 

Jellyfin :

```sh
sudo apt install apt-transport-https
sudo add-apt-repository universe
wget -O - https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | sudo apt-key add -
echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/ubuntu $( lsb_release -c -s ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
sudo apt update && apt install libssl-dev && apt install jellyfin
```
>Start your browser here : http://your-ip:8096



PyLoad Docker ( best ) for download direct Link premium with account 1fichier / uptobox etc.. :

```sh
docker run -d \
  --name=pyload \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -p 8000:8000 \
  -v /home/medias/config:/config \
  -v /home/medias/films:/downloads \
  --restart always \
  ghcr.io/linuxserver/pyload
```


>Documentation : Access the web interface at http://your-ip:8000 the default login is: username - admin password - password


Deluge BitTorrent Client Docker ( Best )/ https://www.newsdemon.com/usenet-access.php : 

```sh
docker run -d \
  --name=deluge \
  --net=host \
  -e UMASK=027 \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Europe/London \
  -e DELUGE_LOGLEVEL=error `#optional` \
  -v /Path/to/config:/config \
  -v /Path/to/films:/downloads \
  --restart always \
  ghcr.io/linuxserver/deluge
```


>Connect on http://your-ip:8112 Password by default is deluge ( change it )
>Set download folder to "/downloads"
>Try Torrent : 
```sh
magnet:?xt=urn:btih:2005C6707977D59A503C92E4E594A282D17EACB0&dn=WandaVision.S01E07.720p.WEB.x265-MiNX%5BTGx%5D+%E2%AD%90&tr=udp%3A%2F%2Ftracker.coppersurfer.tk%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969%2Fannounce&tr=http%3A%2F%2Ftracker.foreverpirates.co%3A80%2Fannounce&tr=udp%3A%2F%2Ftracker.cyberia.is%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.birkenwald.de%3A6969%2Fannounce&tr=udp%3A%2F%2Fexplodie.org%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.opentrackr.org%3A1337%2Fannounce&tr=udp%3A%2F%2Ftracker.tiny-vps.com%3A6969%2Fannounce&tr=udp%3A%2F%2Fexodus.desync.com%3A6969%2Fannounce&tr=udp%3A%2F%2Ftracker.torrent.eu.org%3A451%2Fannounce&tr=udp%3A%2F%2Ftracker.moeking.me%3A6969%2Fannounce&tr=udp%3A%2F%2Fipv4.tracker.harry.lu%3A80%2Fannounce&tr=udp%3A%2F%2Fopen.stealth.si%3A80%2Fannounce&tr=udp%3A%2F%2F9.rarbg.to%3A2710%2Fannounce&tr=udp%3A%2F%2Ftracker.zer0day.to%3A1337%2Fannounce&tr=udp%3A%2F%2Ftracker.leechers-paradise.org%3A6969%2Fannounce&tr=udp%3A%2F%2Fcoppersurfer.tk%3A6969%2Fannounce
```

Add this line if you have a problem in Read and Execute the movies in Jellyfin after download with Deluge.
Every 1 minute, change Read and Execute of file for Jellyfin.
```sh
crontab -e */1 * * * * chmod -R 755 /home/medias/films/ >/dev/null 2>&1
```

 , 




## Installation Nginx

For this part, we need a multiple domain/sub domain. With is is existed sub/domain, to create a rediretion to your IP. Redirection Type A.

Exemple : 
- jellyfin.domain.com
- pyload.domain.com
- torrent.domain.com

The domain.com is your domain :) 


Installation nginx :
```sh
sudo apt install nginx
sudo systemctl start nginx
```

## Reverse Proxy with Nginx

Deluge :
```sh
sudo nano /etc/nginx/conf.d/deluge-webui.conf
```
and copy paste this ( change xxx.domain.com by your domain :
```sh
server {
  listen 80;
  listen [::]:80;
  server_name torrent.yourdomain.com;

  access_log /var/log/nginx/deluge-web.access;
  error_log /var/log/nginx/deluge-web.error;

  location / {
    proxy_pass http://127.0.0.1:8112;
  }
}
```

Restart Nginx :
```sh
sudo nginx -t
sudo systemctl reload nginx
```
Try to connect on torrent.yourdomain.com




Jellyfin :
```sh
sudo nano /etc/nginx/conf.d/jellyfin.conf
```
and copy paste this ( change xxx.domain.com by your domain :
```sh
server {
  listen 80;
  listen [::]:80;
  server_name jellyfin.yourdomain.com;

  access_log /var/log/nginx/jellyfin.access;
  error_log /var/log/nginx/jellyfin.error;

  location / {
    proxy_pass http://127.0.0.1:8096;
  }
}
```

Restart Nginx :
```sh
sudo nginx -t
sudo systemctl reload nginx
```
Try to connect on jellyfin.yourdomain.com

### Etra Jellyfin 

```sh
Extra Jellyfin ( Optional ) :


Cache Video Streams :
#Must be in HTTP block
proxy_cache_path  /home/cache/web levels=1:2 keys_zone=cWEB:50m inactive=90d max_size=35000m;
map $request_uri $h264Level { ~(h264-level=)(.+?)& $2; }
map $request_uri $h264Profile { ~(h264-profile=)(.+?)& $2; }

#set in Server block
proxy_cache cWEB;
proxy_cache_valid 200 301 302 30d;
proxy_ignore_headers Expires Cache-Control Set-Cookie X-Accel-Expires;
proxy_cache_use_stale error timeout invalid_header updating http_500 http_502 http_503 http_504;
proxy_connect_timeout 10s;
proxy_http_version 1.1;
proxy_set_header Connection "";

location /videos/
  {
  proxy_pass http://myJF-IP:8096;
  proxy_cache_key "mydomain.com$uri?MediaSourceId=$arg_MediaSourceId&VideoCodec=$arg_VideoCodec&AudioCodec=$arg_AudioCodec&AudioStreamIndex=$arg_AudioStreamIndex&VideoBitrate=$arg_VideoBitrate&AudioBitrate=$arg_AudioBitrate&SubtitleMethod=$arg_SubtitleMethod&TranscodingMaxAudioChannels=$arg_TranscodingMaxAudioChannels&RequireAvc=$arg_RequireAvc&SegmentContainer=$arg_SegmentContainer&MinSegments=$arg_MinSegments&BreakOnNonKeyFrames=$arg_BreakOnNonKeyFrames&h264-profile=$h264Profile&h264-level=$h264Level";
  proxy_cache_valid 200 301 302 30d;
  }



Cache Images:
# Add this outside of you server block (i.e. http block)
proxy_cache_path /var/cache/nginx/jellyfin levels=1:2 keys_zone=jellyfin:100m max_size=15g inactive=30d use_temp_path=off;

# Cache images (inside server block)
location ~ /Items/(.*)/Images {
  proxy_pass http://127.0.0.1:8096;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_set_header X-Forwarded-Protocol $scheme;
  proxy_set_header X-Forwarded-Host $http_host;

  proxy_cache jellyfin;
  proxy_cache_revalidate on;
  proxy_cache_lock on;
  # add_header X-Cache-Status $upstream_cache_status; # This is only to check if cache is working
}




Rate Limit Downloads:
# Add this outside of you server block (i.e. http block)
limit_conn_zone $binary_remote_addr zone=addr:10m;

# Downloads limit (inside server block)
location ~ /Items/(.*)/Download$ {
   proxy_pass http://127.0.0.1:8096;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_set_header X-Forwarded-Proto $scheme;
   proxy_set_header X-Forwarded-Protocol $scheme;
   proxy_set_header X-Forwarded-Host $http_host;

   limit_rate 1700k; # Speed limit (here is on kb/s)
   limit_conn addr 3; # Number of simultaneous downloads per IP
   limit_conn_status 460; # Custom error handling
   # proxy_buffering on; # Be sure buffering is on (it is by default on nginx), otherwise limits won't work
}

# Error page
error_page 460 http://your-page-telling-your-limit/;

```

Pyload :
```sh
sudo nano /etc/nginx/conf.d/pyload.conf
```
and copy paste this ( change xxx.domain.com by your domain :
```sh
server {
  listen 80;
  listen [::]:80;
  server_name pyload.yourdomain.com;

  access_log /var/log/nginx/pyload.access;
  error_log /var/log/nginx/pyload.error;

  location / {
    proxy_pass http://127.0.0.1:8000;
  }
}
```

Restart Nginx :
```sh
sudo nginx -t
sudo systemctl reload nginx
```
Try to connect on pyload.yourdomain.com

## Installation SSL :
```sh
sudo apt install certbot python3-certbot-nginx

```

Change email and domains
```sh
sudo certbot --nginx --agree-tos --redirect --hsts --staple-ocsp --email you@example.com -d torrent.yourdomain.com -d jellyfin.domain.com -d pyload.domain.com
```

## Release History

* 0.0.1
    * Initial

