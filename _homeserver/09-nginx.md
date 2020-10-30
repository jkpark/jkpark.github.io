---
layout: post
title: 9장. 웹 서버 - nginx
description: 
category: homeserver
---

# Overview

웹 서버는 HTTP 를 사용하여 사용자에게 웹 페이지를 제공해준다.

앞으로 설치할 소프트웨어들의 웹 페이지에 접속하기 위해 웹 서버를 설치해야 한다. 웹 서버는 nginx(엔진엑스)를 골랐다. 예전엔 apache로 많이 설치했는데 기능이 추가되고 추가되면서 무거워졌다. nginx는 apache에서 널리사용되는 기능만 모아 개발되었다. 즉 apache는 현존하는 모든 기능을 사용할 수 있지만 무겁고 nginx는 단순하면서도 성능이 좋다.

> [2018년 11월에 nginx가 apache의 마켓 점유율을 재쳤다.](https://news.netcraft.com/archives/2018/12/17/december-2018-web-server-survey.html#more-26943)


# Nginx 설치

```
$ sudo apt-get install nginx
```

설치된 nginx를 확인한다. 

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

HTTP의 기본 포트인 80번 포트와 HTTPS의 443번 포트의 접근을 허용하도록 한다.

## 공유기의 포트 포워딩

공유기 설정 페이지에서 80번 포트와 443번 포트에 대한 포트포워딩을 해준다.

# 서버블록 설정

Apache에 `VirtualHost`가 있다면 Nginx에는 [Server Block](https://www.nginx.com/resources/wiki/start/topics/examples/server_blocks/)이 있다. `Server Block`은 하나의 머신에서 여러개의 웹 사이트를 운영할 수 있게 해준다. 

`Server Block` 파일은 `/etc/nginx/sites-available`에 저장한다. 이 디렉토리는 말 그래도 사용가능한 서버블록들을 저장하는 경로이고, 실제로 웹 사이트를 활성화시키기 위해선 `/etc/nginx/sites-enabled`에 링크시켜놔야 한다. 

이 포스트에서는 서버블록을 만들고 서비스하는 방법을 다룰 것이다.

`nginx`를 설치하면 `default`라는 서버블록 설정파일이 존재한다. `default` 서버블록은 사용하지 않을 것이므로 `/etc/nginx/sites-enabled`에 링크되어있는 `default`파일을 지운다.

```
$ sudo rm /etc/nginx/sites-enabled/default
```

새로운 서버블록 설정 파일을 만든다. `/etc/nginx/sites-available`에 생성하는 파일명은 자신의 도메인이 `example.com` 이라면 `example.com` 이라고 이름짓는 것을 추천한다. 

```
$ sudo touch /etc/nginx/sites-available/example.com
```

`/etc/nginx/sites-available/example.com` 파일을 열어 아래 내용을 추가한다.

```
$ sudo vi /etc/nginx/sites-available/example.com
```

아래 내용을 그대로 입력한다. **default_server**는 해당 포트에 대한 기본 접속 서버블록을 명시하는 것이다. 도메인이 명시되어 있지않다면 기본 서버블록으로 향하게 된다. 브라우저에서 홈 서버의 IP주소 혹은 호스트명 *example.com* 으로 접속한다면 `/var/www/example.com/htdocs` 안의 `index.html`이 불려질 것이다.

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/example.com/htdocs;

    index index.html;

    server_name example.com;

    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;


    location / {
        try_files $uri $uri/ =404;
    }
}
```

VIM에서 `example.com`을 자신의 도메인으로 치환은 `:%s/example.com/yourdomain.com/gi` 으로 할 수 있다. 

`:wq`로 저장 후 종료한다.

# 서버블록의 index 페이지 작성

웹 사이트의 root 디렉토리를 만들고 index 페이지를 만들어 놓는다.

```
$ sudo mkdir -p /var/www/example.com/htdocs
```

index 페이지를 작성한다.

```
sudo vi /var/www/example.com/htdocs/index.html
```

아래 내용을 입력한 후 저장 후 종료한다.

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

권한을 변경하여 보안성을 높인다.

```
$ sudo chown -R www-data: /var/www/example.com/htdocs
```

# 서버블록 활성화

해당 서버블록을 활성화시키기 위해 링크를 걸어준다.

```
$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

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

HTTPS 통신을 위해서 SSL/TLS 인증서 발급이 필요하다. 앞선 글에서 SSL/TLS 인증서를 발급해주는 Let's Encrypt 에서 SSL/TLS 인증서를 발급받았다.

# DH Param 키 생성

TLS 프로토콜 자체의 기본 설계상 취약점이 존재하므로 TLS용 디피-헬만 키를 사용하여 통신에 안전성을 높인다.

```
$ sudo mkdir -p /usr/local/etc/ssh/certs
$ sudo openssl dhparam -out /usr/local/etc/ssl/certs/dhparam.pem 2048
```

`/etc/nginx/sites-available/example.com` 파일을 열어 모든 내용을 삭제하고 아래 내용을 입력해준다.

```
server {
    listen 80 default_server;
    server_name example.com www.example.com;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$server_name$request_uri;
}
server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

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

    resolver 8.8.8.8 8.8.4.4 valid=300s;

    # include snippets/snakeoil.conf;

    root /var/www/example.com/htdocs;

    index index.html;

    server_name example.com;

    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }
}
```

> 설정은 https://mozilla.github.io/server-side-tls/ssl-config-generator/ 과 https://gist.github.com/jult/395ad9fd3e9773a54a67aaf689beab27 등을 참고하였다.

`ssl_protocols`에 `TLSv1.2`만 허용했는데 `TLSv1.1`이하는 보안에 취약해서 사용하지 않을 것을 권고하고 있다. `resolver`는 인증서 발급업체와 통신하는 네임서버이다. 구글에서 제공하는 네임서버로 셋팅해 주었다.

설정이 끝났으면 `nginx -t`를 통해 문법검사를 하고 재시작한다.

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

브라우저로 http://exsample.com 에 접속하면 자동으로 https://exsample.com 로 리다이렉트된다. 

# 인증서 갱신 후 nginx 재시작

Let’s Encrypt 에서 발급받는 인증서의 경우 유효기간이 90일이고 자동으로 인증서를 갱신하도록 cronjob에 등록되어 있다. 인증서가 갱신되면 nginx를 재시작해주어야 한다.

`/etc/letsencrypt/renewal-hooks/deploy/01-reload-nginx` 파일을 연다.

```
$ sudo vi /etc/letsencrypt/renewal-hooks/deploy/01-reload-nginx
```

아래 내용을 입력하고 저장 후 종료한다.

```
#!/bin/bash
/bin/systemctl reload nginx
```

실행 권한도 추가한다.

```
$ sudo chmod u+x /etc/letsencrypt/renewal-hooks/deploy/01-reload-nginx
```

이제 certbot에 의해 인증서가 갱신되면 `01-reload-nginx` 가 실행되면서 nginx가 재시작될 것이다.
