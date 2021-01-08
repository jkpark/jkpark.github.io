---
title: Plex
description: 
date: 2021-01-08T13:00:05+09:00
draft: false
weight: 3
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Overview

홈 서버의 동영상을 어디에서든 스트리밍 받을 수 있도록 미디어 스트리밍 서버를 설치할 것이다.

# 설치

```
$ echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
$ curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
$ sudo apt-get update
```

`W: Conflicting distribution: https://downloads.plex.tv/repo/deb public InRelease (expected public but got )` 경고는 무시한다. Plex 개발팀에서 고치는 중이라고 한다.


```
$ sudo apt-get install plexmediaserver 
```

설치 도 중 `*** plexmediaserver.list (Y/I/N/O/D/Z) [default=N] ?` 질문에 `N`을 입력한다.


# 방화벽 설정

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 32400 -j ACCEPT
```

# 도메인 설정

plex 미디어 스트리밍을 위해 새로운 서브도메인을 등록할 것이다.

7장에서 생성한 도메인 갱신 스크립트에 서브도메인을 추가하고 실행시키면 도메인 등록이 될 것이다.

```
sudo vi /usr/local/etc/ddns-cloudflare.sh
```

9라인 `A_RECORD_LIST` 에 `plex.example.com` 을 추가한 후 `:wq` 로 저장하고 종료한다.

`-f` 옵션으로 스크립트를 실행 시키면 레코드가 등록된다.

```
$ sudo ./ddns-cloudflare.sh -f
Force update...
2019.03.13_13.52.49 Record example.com is updated.
2019.03.13_13.52.50 Record cloud.example.com is updated.
2019.03.13_13.52.50 Record update failed. Record 'plex.example.com' does not exist.
2019.03.13_13.52.50 Creating a new A record : plex.example.com
2019.03.13_13.52.50 Record plex.example.com is created.
```

# 리버스 프록시

```
$ sudo vi /etc/nginx/sites-available/plex.example.com
```

아래 내용 추가

```
upstream plex_backend {
    server localhost:32400;
    keepalive 32; 
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80; 
    server_name plex.example.com;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name plex.example.com;

    send_timeout 100m; #Some players don't reopen a socket and playback stops totally instead of resuming after an extended pause (e.g. Chrome) 


    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    ssl_dhparam /usr/local/etc/ssl/certs/dhparam.pem;

    ssl_protocols       TLSv1.2;
    ssl_prefer_server_ciphers on; 
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';

    # enable session resumption to improve https performance
    ssl_session_cache shared:SSL:50m;
    ssl_session_timeout 1h; 
    ssl_session_tickets off;

    # fetch OCSP records from URL in ssl_certificate and cache them
    #ssl_stapling on; 
    #ssl_stapling_verify on; 
    # verify chain of trust of OCSP response using Root CA and Intermediate cert
    #ssl_trusted_certificate /etc/nginx/ssl/star_forgott_com.crt;

    # Security Headers
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options nosniff;

    # HSTS (ngx_http_headers_module is required) (31536000 seconds = 1 year)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";

    #Nginx default client_max_body_size is 1MB, which breaks Camera Upload feature from the phones.
    #Increasing the limit fixes the issue. Anyhow, if 4K videos are expected to be uploaded, the size might need to be increased even more
    client_max_body_size 100M;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    root /var/www/plex.example.com/htdocs;

    access_log /var/log/nginx/plex.example.com.access.log;
    error_log /var/log/nginx/plex.example.com.error.log;

    #Plex has A LOT of javascript, xml and html. This helps a lot, but if it causes playback issues with devices turn it off. (Haven't encountered any yet)
    gzip on;
    gzip_vary on;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_types text/plain text/css text/xml application/xml text/javascript application/x-javascript image/svg+xml;
    gzip_disable "MSIE [1-6]\.";

    #Forward real ip and host to Plex
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
        # Plex headers
        proxy_set_header X-Plex-Client-Identifier $http_x_plex_client_identifier;
        proxy_set_header X-Plex-Device $http_x_plex_device;
        proxy_set_header X-Plex-Device-Name $http_x_plex_device_name;
        proxy_set_header X-Plex-Platform $http_x_plex_platform;
        proxy_set_header X-Plex-Platform-Version $http_x_plex_platform_version;
        proxy_set_header X-Plex-Product $http_x_plex_product;
        proxy_set_header X-Plex-Token $http_x_plex_token;
        proxy_set_header X-Plex-Version $http_x_plex_version;
        proxy_set_header X-Plex-Nocache $http_x_plex_nocache;
        proxy_set_header X-Plex-Provides $http_x_plex_provides;
        proxy_set_header X-Plex-Device-Vendor $http_x_plex_device_vendor;
        proxy_set_header X-Plex-Model $http_x_plex_model;

            proxy_set_header        Host                      $server_addr;
            proxy_set_header        Referer                   $server_addr;
            proxy_set_header        Origin                    $server_addr;

    #Websockets
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

        #Disables compression between Plex and Nginx, required if using sub_filter below.
    #May also improve loading time by a very marginal amount, as nginx will compress anyway.
        #proxy_set_header Accept-Encoding "";

    #Buffering off send to the client as soon as the data is received from Plex.
    proxy_redirect off;
    proxy_buffering off;

    location / {
        #Example of using sub_filter to alter what Plex displays, this disables Plex News.
        #sub_filter ',news,' ',';
        #sub_filter_once on;
        #sub_filter_types text/xml;
        proxy_pass http://plex_backend;
    }
}

```

```
$ sudo ln -s /etc/nginx/sites-available/plex.example.com /etc/nginx/sites-enabled/
```

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

# 에이전트 추가

[다음영화](https://movie.daum.net/main/new)에서 한국 영화/드라마 정보를 가져오는 에이전트를 설치한다.

github 주소 : https://github.com/axfree/DaumMovie.bundle

zip파일을 다운로드해서 플러그인 폴더에 압축을 푼다.

```
$ cd /var/lib/plexmediaserver/Library/Application Support/Plex Media Server/Plug-ins
$ sudo wget https://github.com/axfree/DaumMovie.bundle/archive/master.zip
$ unzip master.zip
$ sudo mv DaumMovie.bundle-master DaumMovie.bundle
$ sudo chown -R plex: DaumMovie.bundle
```

시즌17

# 스캐너 추가

한국 드라마 스캐너를 추가한다.

https://github.com/soju6jan/SJVA-Scanners

# 초기 설정

보안상의 이유로 Plex의 초기설정은 로컬호스트 혹은 내부 네트워크에서만 가능하다. 홈 서버는 CLI이기 때문에 윈도우PC에서 SSH 터널링으로 접속한다. SSH터널링은 putty를 이용하였다.

```
$ ssh exsample.com -L 8888:localhost:32400
```
