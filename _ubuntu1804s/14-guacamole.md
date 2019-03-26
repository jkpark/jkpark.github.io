---
layout: post
title: 14장. 웹 기반 원격 접속 클라이언트 - Guacamole
category: ubuntu1804
english: false
---

# Overview

원격 접속 프로그램 중 guacamole은 웹 브라우저를 통해 접속할 수 있다. 가상머신을 웹 브라우저에서 접속하기 위해 guacamole을 설치한다. 

기능 업데이트가 쉬운 docker 기반 컨테이너로 설치할 것이다.


```
$ docker pull guacamole/guacd
$ docker pull guacamole/guacamole
$ docker pull mysql/mysql-server
```

MySql docker를 구동시키고 Guacamole을 위한 DB 생성 스크립트를 생성한다.

```
$ docker run --name=guac-mysqld -d mysql/mysql-server
$ docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```

mysql 패스워드는 아래 명령어로 확인한다.

```
$ docker logs guac-mysqld 2>&1 | grep GENERATED
```

mysql 에 접속한다.

```
$ docker exec -it guac-mysqld mysql -u root -p
```

mysql 컨테이너의 root 비밀번호를 변경한다.

```
> ALTER USER 'root'@'localhost' IDENTIFIED BY '새로운비밀번호';
```

guacamole을 위한 db와 사용자를 생성한다.

```
> CREATE DATABASE guacamole_db;
> CREATE USER `root`@'%' IDENTIFIED BY '비밀번호';
> GRANT ALL PRIVILEGES ON * . * TO 'root'@'%' WITH GRANT OPTION; 
> FLUSH PRIVILEGES;
> quit;
```

init script를 실행한다.

```
$ docker exec -i guac-mysqld mysql guacamole_db -u root -p비밀번호 < initdb.sql
```

guacd를 실행한다.

```
$ docker run --restart always --name guacd -d guacamole/guacd
```

guacamole을 실행한다.

```
$ docker run --restart always --name guacamole \
    --link guacd:guacd \
    --link guac-mysqld:mysql         \
    -e MYSQL_DATABASE=guacamole_db  \
    -e MYSQL_USER=root    \
    -e MYSQL_PASSWORD=qwe \
    -d -p 9095:8080 guacamole/guacamole
```

# 리버스 프록시

```
$ sudo vi /etc/nginx/sites-available/exsample.com
```

```
location = /guacamole {
    return 301 /guacamole/;
}

location /guacamole/ {
    proxy_pass http://127.0.0.1:9095/guacamole/;
    proxy_buffering off;
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
    proxy_cookie_path /guacamole/ /guacamole/;
    access_log off;
}
```

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```


이제 웹 브라우저에서 `https://exsample.com/guacamole` 으로 접속할 수 있다.


guacadmin / guacadmin 로 로그인할 수 있다.


![logined admin account.](/images/ubuntu1804/guacamole01.png)