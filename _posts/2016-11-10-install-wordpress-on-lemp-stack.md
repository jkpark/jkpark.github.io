---
layout: post
title: LEMP stack 에 WordPress 설치방법
description:
category: blog
tags: [MariaDB, NGINX, Ubuntu16.04, LEMP stack, WordPress]
---
Drupal 8 설치 후 테마, 모듈 수가 너무 적어서 Drupal 7을 설치하려고 했지만 WordPress를 설치하여 차이를 느껴보고 싶었다.


[WordPress vs Joomla vs Drupal](http://websitesetup.org/cms-comparison-wordpress-vs-joomla-drupal/)


설치 방법은
<https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu-16-04> 와
<https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-nginx-on-ubuntu-14-04> 를 참고하였다.



# 설치 방법

WordPress를 설치하기 전에 `LEMP stack` 이 설치되어있어야 하고, `SSL`이 활성화되어있어야 한다. 우분투 버전은 16.04 에서 진행하였다.

`LEMP stack` 설치 방법은 [여기](/blog/2016/11/install-lemp-stack-on-ubuntu1604)를 참고.


## WordPress를 위한 Database 생성 및 유저 생성
```bash
jkpark@cactus:/var/www/html$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 1405
Server version: 10.0.27-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database wordpress;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all privileges on wordpress.* to wordpressuser@localhost identified by 'your_passsword';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

## Nginx 설정
```bash
jkpark@cactus:/etc/nginx/sites-available$ sudo vi default
```

```bash
  4     root /var/www/html/wordpress;
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
jkpark@cactus:/var/www/html$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
jkpark@cactus:/var/www/html$ sudo systemctl restart nginx
```

## WordPress 다운로드

`wp-config-sample.php` 샘플설정 파일의 대부분의 설정은 알맞게 되어있어므로 복사하여 기본 설정 파일로 만들어주었다.
```bash
jkpark@cactus:/tmp$ curl -O https://wordpress.org/latest.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 7774k  100 7774k    0     0   465k      0  0:00:16  0:00:16 --:--:--  218k
jkpark@cactus:/tmp$ tar xzvf latest.tar.gz
jkpark@cactus:/tmp$ cp wordpress/wp-config-sample.php wordpress/wp-config.php
```

`upgrade` 디렉토리를 생성하고 `WordPress`를 업그레이드하면 `upgrade` 디렉토리안에 업그레이드에 필요한 파일들이 설치된다.
참고 : <https://www.wpkb.com/wp-files-explained/>
```bash
jkpark@cactus:/tmp$ mkdir /tmp/wordpress/wp-content/upgrade
```

`/tmp/wordpress` 를 `/var/www/html/wordpress` 로 복사한다.

```bash
jkpark@cactus:/tmp/wordpress$ sudo cp -a /tmp/wordpress /var/www/html/wordpress
```

## 소유권과 권한 설정

여기서 하는 작업은 타당한 접근 권한과 소유권을 설정하는 것이다. 일반유저가 파일을 쓸 수 있어야하고, web server가 파일과 디렉토리에 접근, 조작할 수 있어야한다.

파일의 소유권은 `sudo` 유저로 한다.

```bash
jkpark@cactus:/var/www/html$ sudo chown -R jkpark:www-data /var/www/html/wordpress
[sudo] password for jkpark:
jkpark@cactus:/var/www/html$ ls -al
합계 24
drwxr-xr-x 4 www-data www-data 4096 11월  9 21:53 .
drwxr-xr-x 3 root     root     4096 11월  9 14:24 ..
drwxr-xr-x 8 www-data www-data 4096 11월  2 18:08 drupal
-rwxr-xr-x 1 www-data www-data  612 11월  8 22:06 index.nginx-debian.html
-rwxr-xr-x 1 www-data www-data   20 11월  8 23:58 info.php
drwxr-xr-x 5 jkpark   www-data 4096 11월  9 21:44 wordpress
```

각각의 디렉토리에 `setgid` 설정으로 새로 생성되는 파일들은 `www-data` 그룹을 상속받게 한다. 이 설정으로 web server가 생성된 파일들의 group ownership을 가지고 있게 한다.

```bash
jkpark@cactus:/var/www/html/wordpress$ sudo find . -type d -exec chmod g+s {} \;
jkpark@cactus:/var/www/html/wordpress$ ls -al
합계 196
drwxr-sr-x  5 jkpark   www-data  4096 11월  9 21:44 .
drwxr-xr-x  4 www-data www-data  4096 11월  9 21:53 ..
-rw-r--r--  1 jkpark   www-data   418  9월 25  2013 index.php
-rw-r--r--  1 jkpark   www-data 19935  3월  6  2016 license.txt
-rw-r--r--  1 jkpark   www-data  7344  8월 17 05:39 readme.html
-rw-r--r--  1 jkpark   www-data  5456  5월 25 06:02 wp-activate.php
drwxr-sr-x  9 jkpark   www-data  4096  9월  7 23:58 wp-admin
-rw-r--r--  1 jkpark   www-data   364 12월 19  2015 wp-blog-header.php
-rw-r--r--  1 jkpark   www-data  1477  5월 24 01:44 wp-comments-post.php
-rw-r--r--  1 jkpark   www-data  2853 12월 16  2015 wp-config-sample.php
-rw-r--r--  1 jkpark   www-data  2853 11월  9 21:44 wp-config.php
drwxr-sr-x  5 jkpark   www-data  4096 11월  9 21:47 wp-content
-rw-r--r--  1 jkpark   www-data  3286  5월 25  2015 wp-cron.php
drwxr-sr-x 17 jkpark   www-data 12288  9월  7 23:58 wp-includes
-rw-r--r--  1 jkpark   www-data  2382  5월 24 01:44 wp-links-opml.php
-rw-r--r--  1 jkpark   www-data  3353  4월 15  2016 wp-load.php
-rw-r--r--  1 jkpark   www-data 34057  6월 15 06:51 wp-login.php
-rw-r--r--  1 jkpark   www-data  7786  7월 13 21:37 wp-mail.php
-rw-r--r--  1 jkpark   www-data 13920  8월 14 01:02 wp-settings.php
-rw-r--r--  1 jkpark   www-data 29890  5월 25 05:44 wp-signup.php
-rw-r--r--  1 jkpark   www-data  4035 12월  1  2014 wp-trackback.php
-rw-r--r--  1 jkpark   www-data  3064  7월  6 21:40 xmlrpc.php
```


아래 명령어로 group write permission을 부여하여 web interface 가 theme 와 plugin 변경 가능하도록 한다.

```bash
jkpark@cactus:/var/www/html/wordpress$ sudo chmod g+w wp-content
jkpark@cactus:/var/www/html/wordpress$ sudo chmod -R g+w wp-content/themes
jkpark@cactus:/var/www/html/wordpress$ sudo chmod -R g+w wp-content/plugins
jkpark@cactus:/var/www/html/wordpress$
jkpark@cactus:/var/www/html/wordpress$ ls -al wp-content/
합계 24
drwxrwsr-x 5 jkpark www-data 4096 11월  9 21:47 .
drwxr-sr-x 5 jkpark www-data 4096 11월  9 21:44 ..
-rw-r--r-- 1 jkpark www-data   28  1월  9  2012 index.php
drwxrwsr-x 3 jkpark www-data 4096  9월  7 23:59 plugins
drwxrwsr-x 5 jkpark www-data 4096  9월  7 23:59 themes
drwxrwsr-x 2 jkpark www-data 4096 11월  9 21:47 upgrade
```

## WordPress 환경설정

WordPress main configuration 파일을 열었을 때 제일 먼저 해야할 일은 보안키를 설정하는 것이다. `WordPress`가 제공하는 secure generator를 이용하여 키 값을 만들 수 있다.

```bash
jkpark@cactus:/var/www/html/wordpress$ curl -s https://api.wordpress.org/secret-key/1.1/salt/
define('AUTH_KEY',         '(@p -;LQVf1$wxO1 pt&이 글을 복사하지 마세요MhU6+9z2');
define('SECURE_AUTH_KEY',  ')+SoyfkO!HX=vRC(fhb6이 글을 복사하지 마세요Bo&f]KmO');
define('LOGGED_IN_KEY',    'IFBwv/%~3Jwk?(rP,]1C이 글을 복사하지 마세요E ,b]u0V');
define('NONCE_KEY',        '|fHfPA!U,Yh*F:HfZ9[r이 글을 복사하지 마세요eQ?=3>#B');
define('AUTH_SALT',        '|:+NB@<h98|--t)a-dW/이 글을 복사하지 마세요cUx;)x/1');
define('SECURE_AUTH_SALT', 'ZVj.^eMKAtY:5+g:xhu1이 글을 복사하지 마세요T px,TY(');
define('LOGGED_IN_SALT',   'Uv5786a=D%R(~H.zWDJB이 글을 복사하지 마세요Q$XyRWsR');
define('NONCE_SALT',       '$:!yP|#[i8w|FQSBP&&D이 글을 복사하지 마세요4A2=Zp>E');
```

생성된 키들을 `WordPress` 환경설정 파일에 복사한다.

```bash
jkpark@cactus:/var/www/html/wordpress$ vi wp-config.php
```

아래 라인에 위에서 생성한 key들을 붙여넣는다.

```bash
 49 define('AUTH_KEY',         'put your unique phrase here');
 50 define('SECURE_AUTH_KEY',  'put your unique phrase here');
 51 define('LOGGED_IN_KEY',    'put your unique phrase here');
 52 define('NONCE_KEY',        'put your unique phrase here');
 53 define('AUTH_SALT',        'put your unique phrase here');
 54 define('SECURE_AUTH_SALT', 'put your unique phrase here');
 55 define('LOGGED_IN_SALT',   'put your unique phrase here');
 56 define('NONCE_SALT',       'put your unique phrase here');
```

Database 설정

```bash
 23 define('DB_NAME', 'wordpress');
 24
 25 /** MySQL database username */
 26 define('DB_USER', 'wordpressuser');
 27
 28 /** MySQL database password */
 29 define('DB_PASSWORD', '비밀번호');
 30
 31 /** MySQL hostname */
 32 define('DB_HOST', 'localhost');
 33
 34 /** Database Charset to use in creating database tables. */
 35 define('DB_CHARSET', 'utf8');
 36
 37 /** The Database Collate type. Don't change this if in doubt. */
 38 define('DB_COLLATE', '');
```

save and close the file


## Web Interface를 이용하여 설치를 완료한다.

![](https://3.bp.blogspot.com/-FVfQJm5491o/WCMqMy4p24I/AAAAAAAAAiA/vt4KO1sfzHkX9BmeE4lO3njXdGO7CF4uACLcB/s320/%25EC%25BA%25A1%25EC%25B2%25981.PNG)

![](https://3.bp.blogspot.com/-kPW_nb5nwHw/WCMqcBu0jBI/AAAAAAAAAiE/qn8jgNMlxr4HpuHnaa9-uokBYiMJPOGmwCLcB/s320/%25EC%25BA%25A1%25EC%25B2%25982.PNG)



# 업그레이드 방법

`WordPress`의 업그레이드가 가능할 경우 현재 권한으로는 업그레이드를 진행할 수 없다.

위에서 설정한 소유권과 권한으로 보안성이 높아져 소프트웨어가 스스로 update를 완료할 수는 없다.

따라서 업그레이드 시, 임시적으로 web server에게 소유권을 주어 업그레이드를 진행한다.

```bash
sudo chown -R www-data /var/www/html/wordpress
```

업그레이드 완료 시 다시 sudo user에게 소유권을 준다.

```bash
sudo chown -R jkpark /var/www/html/wordpress
```