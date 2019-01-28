---
layout: post
title: LEMP stack 에 Drupal8 설치방법
description:
category: blog
tags: [MariaDB, NGINX, Ubuntu16.04, LEMP stack, Drupal, Ubuntu]
---
<https://www.drupal.org/download> 에서 최신버전의 바이너리를 확인


```bash
jkpark@cactus:/var/www/html$ sudo wget https://ftp.drupal.org/files/projects/drupal-8.2.2.zip
--2016-11-09 14:26:53--  https://ftp.drupal.org/files/projects/drupal-8.2.2.zip
Resolving ftp.drupal.org (ftp.drupal.org)... 151.101.76.68
접속 ftp.drupal.org (ftp.drupal.org)|151.101.76.68|:443... 접속됨.
HTTP request sent, awaiting response... 200 OK
Length: 21205013 (20M) [application/zip]
Saving to: ‘drupal-8.2.2.zip’
drupal-8.2.2.zip            100%[===========================================>]  20.22M  6.32MB/s    in 3.7s
2016-11-09 14:26:58 (5.53 MB/s) - ‘drupal-8.2.2.zip’ saved [21205013/21205013]
```

```bash
jkpark@cactus:/var/www/html$ ls -al
합계 20728
drwxr-xr-x 2 www-data www-data     4096 11월  9 14:26 .
drwxr-xr-x 3 root     root         4096 11월  9 14:24 ..
-rw-r--r-- 1 root     root     21205013 11월  3 03:08 drupal-8.2.2.zip
-rwxr-xr-x 1 www-data www-data      612 11월  8 22:06 index.nginx-debian.html
-rwxr-xr-x 1 www-data www-data       20 11월  8 23:58 info.php
jkpark@cactus:/var/www/html$ sudo unzip drupal-8.2.2.zip
```

```bash
jkpark@cactus:/var/www/html$ sudo mv drupal-8.2.2 drupal
jkpark@cactus:/var/www/html$ ls -al
합계 20732
drwxr-xr-x 3 www-data www-data     4096 11월  9 14:28 .
drwxr-xr-x 3 root     root         4096 11월  9 14:24 ..
drwxr-xr-x 8 root     root         4096 11월  2 18:08 drupal
-rw-r--r-- 1 root     root     21205013 11월  3 03:08 drupal-8.2.2.zip
-rwxr-xr-x 1 www-data www-data      612 11월  8 22:06 index.nginx-debian.html
-rwxr-xr-x 1 www-data www-data       20 11월  8 23:58 info.php
jkpark@cactus:/var/www/html$ sudo rm drupal-8.2.2.zip
```

```bash
jkpark@cactus:/var/www/html$ sudo chown www-data: -R drupal
jkpark@cactus:/var/www/html$ ls -al
합계 20
drwxr-xr-x 3 www-data www-data 4096 11월  9 14:29 .
drwxr-xr-x 3 root     root     4096 11월  9 14:24 ..
drwxr-xr-x 8 www-data www-data 4096 11월  2 18:08 drupal
-rwxr-xr-x 1 www-data www-data  612 11월  8 22:06 index.nginx-debian.html
-rwxr-xr-x 1 www-data www-data   20 11월  8 23:58 info.php
```

데이터베이스 생성
```bash
jkpark@cactus:/var/www/html$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 32
Server version: 10.0.27-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database drupal;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all privileges on drupal.* to drupaluser@localhost identified by 'your_password';
Query OK, 0 rows affected (0.01 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

확인

```bash
jkpark@cactus:/var/www/html$ mysql -u drupaluser -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 34
Server version: 10.0.27-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye
```


drupal 접속을 위해 nginx default 설정파일을 수정한다.

```bash
jkpark@cactus:/etc/nginx/sites-available$ sudo vi default
```

```bash
  4     root /var/www/html/drupal;
  5     index index.html index.htm index.php;
  6     server_name 192.168.0.102;
  7     location / {
  8         index index.php;
  9         try_files $uri $uri/ /index.php$is_args$args;
 10 #try_files $uri $uri/ =404;
 11     }
```


테스트 & 재시작

```bash
jkpark@cactus:/etc/nginx/sites-available$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
jkpark@cactus:/etc/nginx/sites-available$ sudo systemctl restart nginx
```

설정

![](https://2.bp.blogspot.com/-FJXtmhK8BJk/WCK9uFOH_pI/AAAAAAAAAgo/EBFldQj9HPUsHB4VTGUkDZW7f5Q-4nYmACLcB/s500/step1.PNG)

![](https://4.bp.blogspot.com/-hGLa3PTv1-c/WCK9uCCQuBI/AAAAAAAAAgs/D-x5zJkykI82p2HkVBaT3wPvjXaL3QWMQCLcB/s500/step2.PNG)

![](https://3.bp.blogspot.com/--E1mGlUMf18/WCK9uBk87_I/AAAAAAAAAgw/Y86G3AGRh34oYxOyQUQGQTDC8jExPEYiwCLcB/s500/step3.PNG)


php gd모듈과 xml 모듈 설치

```bash
jkpark@cactus:/etc/nginx/sites-available$ sudo apt-get install php7.0-gd php7.0-xml
패키지 목록을 읽는 중입니다... 완료
의존성 트리를 만드는 중입니다
상태 정보를 읽는 중입니다... 완료
다음 새 패키지를 설치할 것입니다:
  php7.0-gd php7.0-xml
0개 업그레이드, 2개 새로 설치, 0개 제거 및 2개 업그레이드 안 함.
139 k바이트 아카이브를 받아야 합니다.
이 작업 후 613 k바이트의 디스크 공간을 더 사용하게 됩니다.
받기:1 http://kr.archive.ubuntu.com/ubuntu xenial-updates/main amd64 php7.0-gd amd64 7.0.8-0ubuntu0.16.04.3 [27.1 kB]
받기:2 http://kr.archive.ubuntu.com/ubuntu xenial-updates/main amd64 php7.0-xml amd64 7.0.8-0ubuntu0.16.04.3 [112 kB]
내려받기 139 k바이트, 소요시간 0초 (583 k바이트/초)
Selecting previously unselected package php7.0-gd.
(데이터베이스 읽는중 ...현재 212758개의 파일과 디렉터리가 설치되어 있습니다.)
Preparing to unpack .../php7.0-gd_7.0.8-0ubuntu0.16.04.3_amd64.deb ...
Unpacking php7.0-gd (7.0.8-0ubuntu0.16.04.3) ...
Selecting previously unselected package php7.0-xml.
Preparing to unpack .../php7.0-xml_7.0.8-0ubuntu0.16.04.3_amd64.deb ...
Unpacking php7.0-xml (7.0.8-0ubuntu0.16.04.3) ...
Processing triggers for php7.0-fpm (7.0.8-0ubuntu0.16.04.3) ...
php7.0-gd (7.0.8-0ubuntu0.16.04.3) 설정하는 중입니다 ...

Creating config file /etc/php/7.0/mods-available/gd.ini with new version
php7.0-xml (7.0.8-0ubuntu0.16.04.3) 설정하는 중입니다 ...

Creating config file /etc/php/7.0/mods-available/dom.ini with new version

Creating config file /etc/php/7.0/mods-available/simplexml.ini with new version

Creating config file /etc/php/7.0/mods-available/wddx.ini with new version

Creating config file /etc/php/7.0/mods-available/xml.ini with new version

Creating config file /etc/php/7.0/mods-available/xmlreader.ini with new version

Creating config file /etc/php/7.0/mods-available/xmlwriter.ini with new version

Creating config file /etc/php/7.0/mods-available/xsl.ini with new version
Processing triggers for php7.0-fpm (7.0.8-0ubuntu0.16.04.3) ...
jkpark@cactus:/etc/nginx/sites-available$ sudo systemctl restart php7.0-fpm.service
```

![](https://4.bp.blogspot.com/-5ppghTTs0eU/WCLAeH5rzaI/AAAAAAAAAhU/qOkylwNc7NAOAJUgR8taHmmVNkziqtEeQCLcB/s500/step4.PNG)

![](https://3.bp.blogspot.com/-bYKd6G15r3k/WCLAePVDs4I/AAAAAAAAAhQ/xshYCQxgqtYZWQ1XSMuZNm19THN8E358gCLcB/s500/step5.PNG)

![](https://1.bp.blogspot.com/-Oo7Zx1GI3ic/WCLAeIj6QhI/AAAAAAAAAhM/f7sfP-BqowEkMLE9m-Qwmjsov4NywJFqgCLcB/s500/step6.PNG)

![](https://1.bp.blogspot.com/-24Qw1piWteY/WCLAed4wzBI/AAAAAAAAAhg/TbdKGaDCeYYkYeGhA2M2euXntlWld3DpQCLcB/s500/step7.PNG)

![](https://1.bp.blogspot.com/-0_hMvm38fa0/WCLAeXPC18I/AAAAAAAAAhY/70qwOW4A-pU-2oapCS4YInQ-kbT_91NDgCLcB/s500/step8.PNG)

![](https://1.bp.blogspot.com/-CDck84-SGbs/WCLAeil2KZI/AAAAAAAAAhc/eHCY8N1NTAsHDnKbE8va6eS78MjQqIlpwCLcB/s500/step9.PNG)

설치 완료.

