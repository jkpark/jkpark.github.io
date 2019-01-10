---
layout: post
title: Ubuntu18.04 - 10 - NextCloud
description: self-hosted cloud storage
category: blog
tags: [Ubuntu, Ubuntu18.04, nextcloud, PHP, NGINX, MariaDB]
english: false
---

> Ubuntu 16.04 에서는 파일공유 프로그램으로 (Resilio Sync)[https://www.resilio.com/individuals/]를 설치했었다. Resilio Sync는 P2P 방식이다. 보통 서버와 스마트폰은 항상 켜놓기 때문에 피어가 살아있어서 언제든지 싱크가 가능했다. 별다른 문제없이 잘 쓰고 있었지만 종종 스마트폰에 부담을 주는 것 같은 느낌을 받았다. 그리고 이사를 하면서 서버를 꽤 오랜시간동안 방치했고 그 기간동안 스마트폰도 바꾸고 윈도우PC의 파일도 정리하고 이것저것 만지다보니 싱크가 어디서부터 꼬였는지 알 수 없었다. 트러블슈팅도 어려웠고 이번에 우분투 18.04를 설치하게 되면서 새로운 파일 싱크 툴을 찾게 되었다.

[NextCloud](https://nextcloud.com/)는 Opensource 이며 Self-hosting가 가능하고 윈도우, 리눅스, 안드로이드 모두 지원한다. 또한 이미 LEMP Stack이 완성되어 있기 때문에 쉽게 설치할 수 있다.

# Overview

NextCloud를 설치는 nginx, php, mariadb 를 기반으로 할 것이다. [nginx설치](install_ubuntu1804-08-nginx)와 [php and mariddb 설치](install_ubuntu1804-09-php-and-mariadb)를 마쳤다는 전제 조건에서 진행한다.


# Prerequisites

> 어떤 PHP 모듈이 필요한지는 [이 페이지](https://docs.nextcloud.com/server/15/admin_manual/installation/source_installation.html#prerequisites-for-manual-installation)를 참고하면 된다.

NextCloud 설치에 필요한 필수 php 모듈을 설치한다.

```
$ sudo apt-get install php7.2-curl php7.2-gd php7.2-json php7.2-xml php7.2-mbstring php7.2-zip
```

추천 모듈도 설치한다.

```
$ sudo apt-get install php7.2-intl php-apcu
```

# NextCloud 설치파일 다운로드

NextCloud archive 파일을 다운받아 압축해체하고 nextcloud의 document-root가 될 디렉토리로 이동시킨다.

```
$ wget https://download.nextcloud.com/server/releases/latest.tar.bz2
$ tar xjf latest.tar.bz2
$ sudo mv nextcloud /var/www/
$ sudo chown -R www-data: /var/www/nextcloud
```

# Nginx 서버블록 설정

`/etc/nginx/sites-available/cloud.cactus.conf`라는 이름으로 새로운 서버블록 설정파일을 만들었다.
> 참고 : https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html#nextcloud-in-the-webroot-of-nginx

```
##
server {
    listen 88;
    server_name cactus;
    return 301 https://$server_name:10443$request_uri;
}
server {
    listen 10443 ssl http2;
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
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;
    add_header Referrer-Policy no-referrer;

	# HSTS (ngx_http_headers_module is required) (31536000 seconds = 1 year)
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";

	# Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

	resolver 10.32.192.11 10.32.193.11 valid=300s;

	# include snippets/snakeoil.conf;

	root /var/www/nextcloud;

	location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

	location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }

	# set max upload size
    client_max_body_size 102400M;
    fastcgi_buffers 64 4K;

	# Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php$request_uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
		fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js and css files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff2?|svg|gif)$ {
        try_files $uri /index.php$request_uri;
        add_header Cache-Control "public, max-age=15778463";
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        add_header Referrer-Policy no-referrer;

        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
        try_files $uri /index.php$request_uri;
        # Optional: Don't log access to other assets
        access_log off;
    }

	access_log /var/log/nginx/cloud.cactus.access.log;
    error_log /var/log/nginx/cloud.cactus.error.log;
}
```

`/etc/nginx/sites-enabled` 에 링크걸어준다.

```
$ sudo ln -s /etc/nginx/sites-available/cloud.cactus.conf /etc/nginx/sites-enabled/
```

설정이 끝났으면 문법검사를 하고 재시작한다.

```
sudo nginx -t
sudo systemctl restart nginx
```

# PHP 설정

`client_max_body_size`를 100GB로 잡았기 때문에 관련 php 설정도 바꿔줘야 한다.
`/etc/php/7.2/fpm/php.ini` 파일에 아래 내용을 수정한다.

```
post_max_size = 110000M
upload_max_filesize = 100G
max_execution_time = 3600
max_input_time = 3600
memory_limit = 512M
```

`post_max_size` : 하나의 게시물의 최대 사이즈
`max_execution_time` : php 스크립트의 실행시간. 파일 업로드하는데 소요되는 시간 포함
`max_input_time` : 각 스크립트가 데이터 파싱을 하는 시간. 파일 업로드하는데 소요되는 시간 포함
`memory_limit` : 스크립트가 사용하는 메모리 크기

`/etc/php/7.2/fpm/pool.d/www.conf` 파일을 열어 아래 내용을 수정한다.

```
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

PHP 도 재시작한다.

```
$ sudo systemctl restart php7.2-fpm.service
```

# Database 설정

`$ sudo mariadb -u root -p` 로 DB에 접속한 다음 아래 명령어를 통해 nextcloud 에서 사용할 데이터베이스를 만든다.

```
> CREATE DATABASE nextcloud_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
> create user 'ncuser'@'localhost' identified by '비밀번호';
> grant all privileges on nextcloud_db.* to 'ncuser'@'localhost';
> flush privileges;
> exit;
```

`/etc/mysql/mariadb.conf.d/50-server.cnf` 파일을 수정한다.

```
[mysqld]
transaction-isolation = READ-COMMITTED

innodb_large_prefix = true
innodb_file_format = barracuda
innodb_file_per_table = 1
```

# NextCloud Installation wizard

브라우저를 통해 nextcloud 서버블록(`https://cactus:10443`)으로 접속하면 아래와 같은 위자드 창이 뜬다.

![](/images/posts/install-ubuntu1804/nextcloud-index.png)

`사용자 이름` : 생성할 관리자 계정
`암호` : 관리자 계정의 암호
`데이터 폴더` : 업로드 데이터가 저장될 폴더


# Tips and tricks

`/var/www/nextcloud/.user.ini` 파일을 열어 `upload_max_filesize`, `post_max_size`, `memory_limit`을 지워준다.


`/var/www/nextcloud/config/config.php` 파일을 열어 APCu 옵션을 추가한다.

```
'memcache.local' => '\OC\Memcache\APCu',
```