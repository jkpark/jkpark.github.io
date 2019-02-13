---
layout: post
title: 08 - nginx
description: 
category: ubuntu1804
english: false
---

# Overview

[LEMP Stack](https://lemp.io/) 이란 Linux, Nginx, MariaDB, PHP를 조합하여 줄인말이다.

[2018년 11월에 드디어 nginx가 apache의 마켓 점유율을 재쳤다.](https://news.netcraft.com/archives/2018/12/17/december-2018-web-server-survey.html#more-26943)

이 포스트에서는 nginx 설치에 대해 다룬다. php와 mariadb는 [다음 포스트](09-php-and-mariadb)를 참고하기 바란다.

# Nginx Installation

```
$ sudo apt-get install nginx
```

```
$ nginx -v
nginx version: nginx/1.14.0 (Ubuntu)
```

# 방화벽 설정

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

# 웹 사이트의 root 디렉토리 설정

웹 사이트의 root 디렉토리를 만들고 index 페이지를 만들어놓는다.

```
$ sudo mkdir -p /var/www/cactus/htdocs
$ sudo chown -R www-data: /var/www/cactus
```

`sudo vi /var/www/cactus/htdocs/index.html`

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
</head>
<body>
    it worked.
</body>
</html>
```

# 서버 블록 설정

Apache에 `VirtualHost`가 있다면 Nginx에는 [Server Block](https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/)이 있다. `Server Block`은 하나의 머신에서 여러개의 웹 사이트를 운영할 수 있게 해준다. 

`Server Block` 파일은 `/etc/nginx/sites-available`에 저장한다. 이 디렉토리는 말 그래도 사용가능한 서버블록들을 저장하는 경로이고, 실제로 웹 사이트를 활성화시키기 위해선 `/etc/nginx/sites-enabled`에 링크시켜놔야 한다. `nginx`를 설치하면 `default`라는 서버블록 설정파일이 존재한다.  `/etc/nginx/sites-enabled`에 링크되어있는 `default`파일을 지우고 새로운 서버블록 설정 파일을 만든다. `/etc/nginx/sites-available`에 생성하는 파일명은 자신의 도메인이 `exsample.com` 이라면 `exsample.com.conf` 이라고 이름짓는 것을 추천한다. 

나는 `cactus`라는 이름으로 설정하였다.

```
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo touch /etc/nginx/sites-available/cactus.conf
$ sudo ln -s /etc/nginx/sites-available/cactus.conf /etc/nginx/sites-enabled/cactus.conf
```

`/etc/nginx/sites-available/cactus.conf` 파일을 열어 아래 내용을 추가한다.

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/cactus/htdocs;

    index index.html;

    server_name cactus;

    access_log /var/log/nginx/cactus.access.log;
    error_log /var/log/nginx/cactus.error.log;


    location / {
        try_files $uri $uri/ =404;
    }


}
```

`default_server`는 해당 포트에 대한 기본 접속 서버블록을 명시하는 것이다. 도메인이 명시되어 있지않다면 기본 서버블록으로 향하게 된다.

`nginx -t` 명령을 통해 설정에 이상이 없는지 확인할 수 있다.

```
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

이상이 없다면 nginx를 재시작한다.

```
$ sudo systemctl restart nginx
```

브라우저를 통해 자신의 웹 사이트에 접속이 되는지 확인해본다.

# HTTPS 활성화

HTTPS 통신을 위해서 함호화하는 key가 필요하다. 도메인을 가지고 있다면 무료로 SSL/TLS 인증서를 발급해주는 Let's Encrypt 에서 key를 발급받으면 된다. 나는 외부에서 접속을 허용하지 않을 것이기 때문에 로컬에서 key를 만들었다.

```
$ sudo mkdir -p /etc/nginx/ssl/cactus
$ sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/nginx/ssl/cactus/nginx.key -out /etc/nginx/ssl/cactus/nginx.crt
```

`/etc/nginx/sites-available/cactus.conf` 파일을 열어 모든 내용을 삭제하고 아래 내용을 입력해준다.

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name cactus;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name cactus;

    # SSL configuration
    ssl_certificate     /etc/nginx/ssl/cactus/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/cactus/nginx.key;
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
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    resolver 10.32.192.11 10.32.193.11 valid=300s;

    # include snippets/snakeoil.conf;

    root /var/www/cactus/htdocs;

    index index.html;

    access_log /var/log/nginx/cactus.access.log;
    error_log /var/log/nginx/cactus.error.log;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }
}
```

> 설정은 https://mozilla.github.io/server-side-tls/ssl-config-generator/ 과 https://gist.github.com/jult/395ad9fd3e9773a54a67aaf689beab27 등을 참고하였다.

`ssl_protocols`에 `TLSv1.2`만 허용했는데 `TLSv1.1`이하는 보안에 취약해서 사용하지 않을 것을 권고하고 있다. `TLSv1.3` 버전도 2018년 8월에 릴리즈되었다. `TLSv1.3`를 사용하기 위해선 `OpenSSL 1.1.1` 이상의 버전으로 업그레이드 해야한다. 우분투 18.04에는 `OpenSSL 1.1.0g`가 설치되어 있기 때문에 추 후 업그레이드를 진행할 것이다.

설정이 끝났으면 `nginx -t`를 통해 문법검사를 하고 재시작한다.

```
sudo nginx -t
sudo systemctl restart nginx
```

브라우저로 http://cactus 에 접속하면 자동으로 https://cactus 로 리다이렉트된다. 로컬에서 만든 인증서로 SSL연결을 하기 때문에 아래와 같은 경고가 뜰 것이다.

![](/images/posts/install-ubuntu1804/lempstack-https.png)

`Advanced` 버튼을 눌러 `Proceed to cactus`를 클릭하면 접속된다.
