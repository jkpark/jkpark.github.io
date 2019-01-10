---
layout: post
title: Ubuntu18.04 - 09 - PHP and MariaDB
description: 
category: blog
tags: [Ubuntu, Ubuntu18.04, PHP, MariaDB]
english: false
---

# Overview

[LEMP Stack](https://lemp.io/) 이란 Linux, Nginx, MariaDB, PHP를 조합하여 줄인말이다.

[2018년 11월에 드디어 nginx가 apache의 마켓 점유율을 재쳤다.](https://news.netcraft.com/archives/2018/12/17/december-2018-web-server-survey.html#more-26943)

이 포스트에서는 php, mariadb 설치에 대해 다룬다. nginx 설치에 대해서는 [지난 포스트](install_ubuntu1804-08-nginx)에서 다뤘다.

# PHP Installation

php를 설치하면 apache가 함께 설치된다. apache를 설치하지 않으려면 php-fpm을 먼저 설치하고 php을 설치하면 apache가 설치되지 않는다.

```
$ sudo apt-get install php-fpm
$ sudo apt-get install php
```

php 기본 설정파일을 열어 아래 옵션을 바꿔준다.

```
mbstring.language = UTF-8
mbstring.internal_encoding = UTF-8
mbstring.http_input = UTF-8
mbstring.http_output = UTF-8
```

`/etc/nginx/sites-available/cactus.conf` 서버 블록 파일에서 php에 대한 설정을 해준다. 

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

    index index.php index.html;

    access_log /var/log/nginx/cactus.access.log;
    error_log /var/log/nginx/cactus.error.log;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
    #   # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
    }

    client_max_body_size 0;
}
```

`index` 부분에 `index.php`을 먼저 로드하도록 선언하였고, `location ~ \.php$`에  `FastCGI`에 대한 설정도 하였다. `client_max_body_size 0`은 요청 body의 크기 제한을 disable하는 것이다. 파일 업로드가 필요할 경우 body size가 제한되어 있다면 에러가 날 것이다.

nginx 설정을 추가하였으니 문법 확인 및 재시작을 해준다.

```
sudo nginx -t
sudo systemctl restart nginx
```

`cactus` 서버 블록의 root 디렉토리인 `/var/www/cactus/htdocs`에 php의 정보를 볼 수 있는 페이지 `info.php`를 만든다. 이 페이지를 보고 싶다면 `https://cactus/info.php`를 요청하면 된다.

```
$ sudo su -c 'echo "<?php phpinfo(); ?>" |tee /var/www/cactus/htdocs/info.php'
```

![](/images/posts/install-ubuntu1804/phpinfo.png)

# MariaDB 설치

```
$ sudo apt-get install mariadb-server php7.2-mysql
```

`php7.2-mysql` 는 php7.2에서 mariadb 사용을 위한 모듈이다. mariadb가 mysql에서 파생된 프로젝트이기 때문에 php7.2-mysql을 설치한다.

`mysql_secure_installation` 을 실행하여 기본 설정을 진행한다.

```
$ sudo mysql_secure_installation
Enter current password for root (enter for none):
Set root password? [Y/n] y
New password:
Re-enter new password:
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

mariadb에 접속해본다.

```
$ sudo mariadb -u root -p
```

`/etc/mysql/mariadb.conf.d/50-server.cnf` 파일을 열어 기본 언어셋은 바꿔준다.
기본언어셋으로 `utf8mb4_general_ci`가 설정되어 있는데 한글 정렬이 잘 안되는 문제가 있다. 이것을 `utf8mb4_unicode_ci`로 바꿔준다.
> 참고 : https://stackoverflow.com/questions/766809/whats-the-difference-between-utf8-general-ci-and-utf8-unicode-ci

```
collation-server      = utf8mb4_unicode_ci
```

빠져나올 때는 `exit;`을 하면 종료된다.

- - -

이로써 LEMP Stack이 완성되었다.
