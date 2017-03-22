---
layout: post
title: How To Install Nginx, MariaDB10, PHP7(LEMP stack) on Ubuntu16.04
description:
category: blog
tags: [MariaDB, NGINX, PHP, LEMP stack, Ubuntu16.04]
---
[LEMP Stack](https://lemp.io/) 이란

Linux, Nginx, MariaDB, PHP를 조합하여 줄인말이다.

`Apache` vs `Nginx`

`Nginx`(엔진X)는 `Apache`의 잘 사용하지 않는 기능을 제외하고 새로운 시대의 요청에 부응하여 만들어진 웹 서버이다.

[PHP 속도 테스트](http://www.theorganicagency.com/blog/apache-vs-nginx-performance-comparison/)

![](https://3.bp.blogspot.com/-i3piYEtcSso/WCHK14sRsSI/AAAAAAAAAfY/A3W4qePtFdYxFBaMcBAAPigTHlm1qlqIACLcB/s400/%25EC%25BA%25A1%25EC%25B2%2598.PNG)
https://news.netcraft.com/archives/2016/02/22/february-2016-web-server-survey.html

[MariaDB](https://ko.wikipedia.org/wiki/MariaDB) 는 MySQL의 불확실한 라이선스 상태에 반발하여 만들어졌으며 MariaDB 5.5버전까지는 모든 MySQL의 특징을 가지고 있었다.
5.5버전 이후 MariaDB만의 색깔을 더 분명히 하고자 개발판을 10.x로 변경하였다.


#### Nginx Web Server 설치

install Nginx

```bash
jkpark@cactus:~$ sudo apt-get install nginx
[sudo] password for jkpark:
패키지 목록을 읽는 중입니다... 완료
의존성 트리를 만드는 중입니다
상태 정보를 읽는 중입니다... 완료
The following additional packages will be installed:
  nginx-common nginx-core
제안하는 패키지:
  fcgiwrap nginx-doc
다음 새 패키지를 설치할 것입니다:
  nginx nginx-common nginx-core
0개 업그레이드, 3개 새로 설치, 0개 제거 및 2개 업그레이드 안 함.
458 k바이트 아카이브를 받아야 합니다.
이 작업 후 1,482 k바이트의 디스크 공간을 더 사용하게 됩니다.
계속 하시겠습니까? [Y/n] y
받기:1 http://kr.archive.ubuntu.com/ubuntu xenial-updates/main amd64 nginx-common all 1.10.0-0ubuntu0.16.04.4 [26.6 kB]
받기:2 http://kr.archive.ubuntu.com/ubuntu xenial-updates/main amd64 nginx-core amd64 1.10.0-0ubuntu0.16.04.4 [428 kB]
받기:3 http://kr.archive.ubuntu.com/ubuntu xenial-updates/main amd64 nginx all 1.10.0-0ubuntu0.16.04.4 [3,498 B]
내려받기 458 k바이트, 소요시간 0초 (1,224 k바이트/초)
패키지를 미리 설정하는 중입니다...
Selecting previously unselected package nginx-common.
(데이터베이스 읽는중 ...현재 212073개의 파일과 디렉터리가 설치되어 있습니다.)
Preparing to unpack .../nginx-common_1.10.0-0ubuntu0.16.04.4_all.deb ...
Unpacking nginx-common (1.10.0-0ubuntu0.16.04.4) ...
Selecting previously unselected package nginx-core.
Preparing to unpack .../nginx-core_1.10.0-0ubuntu0.16.04.4_amd64.deb ...
Unpacking nginx-core (1.10.0-0ubuntu0.16.04.4) ...
Selecting previously unselected package nginx.
Preparing to unpack .../nginx_1.10.0-0ubuntu0.16.04.4_all.deb ...
Unpacking nginx (1.10.0-0ubuntu0.16.04.4) ...
Processing triggers for ufw (0.35-0ubuntu2) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for systemd (229-4ubuntu12) ...
nginx-common (1.10.0-0ubuntu0.16.04.4) 설정하는 중입니다 ...
nginx-core (1.10.0-0ubuntu0.16.04.4) 설정하는 중입니다 ...
nginx (1.10.0-0ubuntu0.16.04.4) 설정하는 중입니다 ...
Processing triggers for systemd (229-4ubuntu12) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for ufw (0.35-0ubuntu2) ...
```


`netstat`, `systemctl` 또는 브라우저로 `nginx` 동작 확인 (디폴트 포트80)

```bash
jkpark@cactus:~$ sudo systemctl status nginx.service
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since 화 2016-11-08 22:06:57 KST; 3min 41s ago
 Main PID: 16028 (nginx)
   CGroup: /system.slice/nginx.service
           ├─16028 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           ├─16029 nginx: worker process
           ├─16030 nginx: worker process
           ├─16031 nginx: worker process
           └─16032 nginx: worker process

11월 08 22:06:57 cactus systemd[1]: Starting A high performance web server and a reverse proxy server...
11월 08 22:06:57 cactus systemd[1]: Started A high performance web server and a reverse proxy server.
```

![](https://4.bp.blogspot.com/-SGqaJImaZfk/WCHPFH7FmwI/AAAAAAAAAfs/dEsIz4mBFYMG8V9kU94Kr3oGUTTxWarHwCLcB/s320/%25EC%25BA%25A1%25EC%25B2%2598.PNG)



#### HTTP/2.0 프로토콜 활설화

lastest release of Nginx binaries의 `HTTP/2.0` 프로토콜은 `SSL`과 함께 사용해야 동작된다.

```bash
jkpark@cactus:/etc/nginx/sites-available$ sudo mv default default.backup
jkpark@cactus:/etc/nginx/sites-available$ sudo vi default
```

아래 내용 복사하여 `HTTP/2.0` 을 활성화한다.

`server_name`는 자신의 IP주소를 적는다.

```
server {
listen 443 ssl http2 default_server;
listen [::]:443 ssl http2 default_server;
root /var/www/html;
index index.html index.htm index.php;
server_name 192.168.0.102;
location / {
try_files $uri $uri/ =404;
}
ssl_certificate /etc/nginx/ssl/nginx.crt;
ssl_certificate_key /etc/nginx/ssl/nginx.key;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
ssl_dhparam  /etc/nginx/ssl/dhparam.pem;
ssl_session_cache shared:SSL:20m;
ssl_session_timeout 180m;
resolver 8.8.8.8 8.8.4.4;
add_header Strict-Transport-Security "max-age=31536000;
#includeSubDomains" always;
location ~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/run/php/php7.0-fpm.sock;
}
location ~ /\.ht {
deny all;
}
}
server {
listen         80;
listen    [::]:80;
server_name    192.168.0.102;
return         301 https://$server_name$request_uri;
}
```



위의 default 설정이 끝나면 `SSL` 사용을 위한 파일을 만든다.

```bash
jkpark@cactus:/etc/nginx$ sudo mkdir ssl
jkpark@cactus:/etc/nginx$ sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
Generating a 2048 bit RSA private key
..........................................................+++
............................................................................................+++
writing new private key to '/etc/nginx/ssl/nginx.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:KR
State or Province Name (full name) [Some-State]:.
Locality Name (eg, city) []:Seoul
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Indiecoders
Organizational Unit Name (eg, section) []:.
Common Name (e.g. server FQDN or YOUR name) []:.
Email Address []:jkparkr@indiecoders.org
jkpark@cactus:/etc/nginx$

5. DH(Diffie-Hellman)암호키를 만든다.
jkpark@cactus:/etc/nginx$ sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.............................................................++*++*
```

DH암호키가 만들어졌다면 `Nginx` 설정이 제대로 되었는지 확인해본다.

```bash
jkpark@cactus:/etc/nginx$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
jkpark@cactus:/etc/nginx$ sudo systemctl restart nginx.service
```

아래 명령어로 `Nginx`가 `HTTP/2.0` 프로토콜을 사용하고 있는지 확인할 수 있다.

```bash
jkpark@cactus:/etc/nginx$ openssl s_client -connect localhost:443 -nextprotoneg ''
CONNECTED(00000003)
Protocols advertised by server: h2, http/1.1
140205134509720:error:140920E3:SSL routines:ssl3_get_server_hello:parse tlsext:s3_clnt.c:1152:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 87 bytes and written 7 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
Next protocol: (2)
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1478612682
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
```

#### PHP7 설치

Nginx w/ FastCGI 세팅을 위해 `PHP7`과 `PHP-FPM`을 설치한다.

```bash
jkpark@cactus:~$ sudo apt install php7.0 php7.0-fpm
```

설치가 끝나면 `PHP-FPM`을 실행하고 체크해본다.

```bash
jkpark@cactus:~$ sudo systemctl start php7.0-fpm
jkpark@cactus:~$ sudo systemctl status php7.0-fpm
● php7.0-fpm.service - The PHP 7.0 FastCGI Process Manager
   Loaded: loaded (/lib/systemd/system/php7.0-fpm.service; enabled; vendor preset: enabled)
   Active: active (running) since 화 2016-11-08 23:47:13 KST; 1min 14s ago
 Main PID: 23164 (php-fpm7.0)
   Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
   CGroup: /system.slice/php7.0-fpm.service
           ├─23164 php-fpm: master process (/etc/php/7.0/fpm/php-fpm.conf)
           ├─23168 php-fpm: pool www
           └─23169 php-fpm: pool www

11월 08 23:47:13 cactus systemd[1]: Starting The PHP 7.0 FastCGI Process Manager...
11월 08 23:47:13 cactus systemd[1]: Started The PHP 7.0 FastCGI Process Manager.
```

위에서 설정한 `Nginx` `default` 설정 파일에 아래 구문을 통해 이미 `PHP FastCGI`에 대한 설정을 하였다.

```bash
location ~ \.php$ {
include snippets/fastcgi-php.conf;
fastcgi_pass unix:/run/php/php7.0-fpm.sock;
}
```

아래 명령어를 통해 `info.php` 파일을 만들어 `PHP` 설정을 확인할 수 있다.

```bash
jkpark@cactus:~$ sudo su -c 'echo "<?php phpinfo(); ?>" |tee /var/www/html/info.php'
<?php phpinfo(); ?>
```

![](https://1.bp.blogspot.com/-yU3yEJ54-4s/WCHoi_YParI/AAAAAAAAAgE/hb4hzWi-TP47vHEgOwCzfBK_OAicLx_LQCLcB/s400/%25EC%25BA%25A1%25EC%25B2%2598.PNG)


또한 `$_SERVER['SERVER_PROTOCOL']` 항목에서 `HTTP/2.0` 프로토콜이 동작중인지 확인할 수 있다.

`apt search php7.0` 명령어를 통해 모듈을 검색할 수 있고 `sudo apt install php7.0-****` 을 통해 모듈을 설치하였다면 `PHP-FPM` 데몬을 재시작하여 설치된 모듈을 등록할 수 있다.

```bash
sudo systemctl restart php7.0-fpm.service
```

#### MariaDB 설치

미지막으로 MariaDB를 설치하여 LEMP Stack을 완성시키자.

```bash
jkpark@cactus:~$ sudo apt install mariadb-server mariadb-client php7.0-mysql
```

설치 후 `PHP-RPM` 데몬 재시작한다.

```bash
jkpark@cactus:~$ sudo systemctl restart php7.0-fpm.service
```

`mysql_secure_installation` 스크립트를 실행하여 `MariaDB`의 root password를 설정한다.

```
jkpark@cactus:~$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

일반유저가 `sudo` 특권없이 `Database`에 접근하기 위해 아래 명령어를 입력한다.

```bash
jkpark@cactus:~$ sudo mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 48
Server version: 10.0.27-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> update user set plugin='' where User='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [mysql]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [mysql]> exit
Bye
```

이제 `sudo` 없이 `mysql`에 접근할 수 있다.

```bash
jkpark@cactus:~$ mysql -u root -p -e 'show databases'
Enter password:
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

언어셋 설정
기본언어셋으로 `utf8mb4_general_ci`가 설정되어 있는데 이것을 `utf8mb4_unicode_ci`로 바꿔준다.

[difference between utf8_general_ci and utf8_unicode_ci](http://stackoverflow.com/questions/766809/whats-the-difference-between-utf8-general-ci-and-utf8-unicode-ci)

```bash
jkpark@cactus:/etc/mysql/mariadb.conf.d$ sudo vi 50-server.cnf
```

```
105 character-set-server  = utf8mb4
106 #original
107 #collation-server      = utf8mb4_general_ci
108 #jkpark modified 2016-11-09
109 collation-server      = utf8mb4_unicode_ci
```


설정이 끝나면` mysql` 데몬을 제시작하고 아래 명령어로 언어셋을 확인한다.

```bash
MariaDB [(none)]> show variables like "%character%";show variables like "%collation%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_general_ci |
| collation_database   | utf8mb4_unicode_ci |
| collation_server     | utf8mb4_unicode_ci |
+----------------------+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]>
```

* `collation_connection 은 utf8mb4_unicode_ci` 로 설정이 안되는데 아시는분은 답글부탁드립니다.

이상으로 `LEMP Stack` 설치가 끝났다.