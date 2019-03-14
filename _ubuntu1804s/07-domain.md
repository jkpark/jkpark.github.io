---
layout: post
title: 7장. 도메인
description:
category: ubuntu1804
---

# Overview

도메인은 네트워크 상에서 서버에 접속할 때 기억하기 어려운 IP주소 대신 exsample.com 같은 쉬운 이름으로 접속하는 주소다. 브라우저에서 exsample.com 에 접속을 요청하면 DNS(Domain Name System)서버의 DNS레코드를 보고 이에 해당하는 IP주소로 연결한다.

이해를 돕기 위해 아래 동영상을 시청하길 바란다.

<iframe width="602" height="295" src="https://www.youtube.com/embed/2ZUxoi7YNgs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# 도메인 대여

도메인을 이용하기 위해선 대체로 연간 1~2만원 정도를 지불 해야한다. 이 경우 해당 도메인을 소유할 수 있고 법적인 보호를 받는다. 무료 도메인을 제공해주는 기업도 있다. [freenom](www.freenom.com) 에서 `.TK`, `.ML`, `.GA`, `.CF`, .`GQ` 를 무료로 제공하지만 소유권은 해당 도메인 제공자에게 있기 때문에 법적인 보호를 받지 못한다.

> `.TK`, `.ML`, `.GA`, `.CF`, .`GQ` 도 지불한다면 소유할 수 있다. https://www.freenom.com/en/freeandpaiddomains.html

# DNS 레코드
도메인은 www.exsample.com 이런 식으로 되어 있는데 `.com`은 최상위 도메인이다. www 없이 `exsample.com` 만 있는 도메인을 루트 도메인이라고 하고, `www.exsample.com` 혹은 `m.exsample.com` 같은 접두어가 붙는 도메인을 서브 도메인이라고 한다. 서브 도메인은 DNS 레코드 설정에서 생성하고 삭제할 수 있다.

DNS 레코드 타입에는 여러 종류가 있다. 중요한 3 가지만 살펴본다.

### A (Address Mapping records)

주어진 IP주소로 연결한다. 주어지는 이름이 `@` 라면 루트 도메인을 의미하고 `www`라면 www.exsample.com 을 의미한다.

### CNAME (Canonical Name)

도메인 이름을 다른 도메인으로 매핑시킨다. 

### TXT (Text)

임의의 텍스트 문자열을 저장할 수 있다. 보안 문서를 지정할 때 쓰인다.

DNS 레코드 작성도 도메인을 등록한 www.freenom.com 에서 할 수 있지만, DDNS를 이용하기 위해 `CloudFlare`의 DNS을 이용할 것이다. 도메인의 네임서버를 `CloudFlare`의 네임서버로 변경하고 DNS 레코드도 작성한다.

# DDNS

가정에서 운용할 홈 서버의 IP주소는 가변적이다. `DDNS`는 IP주소가 변동되면 IP주소를 도메인을 관리하는 네임서버에 다시 연결해주는 것을 말한다. 구글 도메인 같이 대형 도메인 등록 업체은 자체적으로 DDNS 서비스를 제공하기 때문에 해당 기관의 DDNS 서비스 이용방법을 살펴보면 된다. 도메인 등록 업체에서 DDNS 서비스를 제공하지 않는다면 DDNS 서비스를 이용할 수 있는 DNSZi, CloudFlare 등을 통해 DDNS 서비스를 적용할 수 있다.

무료 도메인 `.ga` 같은 경우 국내 업체인 `DNSZi`에서는 DNS사용을 제한하고 있기 때문에 `CloudFlare`를 선택했다. CloudFlare의 경우 쉘 스크립트로 DNS레코드를 업테이트 할 수 있기 때문에 주기적으로 DNS 레코드의 IP와 홈 서버의 IP가 다른지 체크하고 DNS 레코드를 업데이트 함으로 DDNS 기능을 이용할 수 있다.

쉘 스크립트 작성에 앞 서 자신의 `Global APK Key`를 확인해야한다.

CloudFlare 웹사이트에서 `My Profile` -> `API Keys` 에서 `Global API Key` 를 확인한다.

![Global API Key](/images/ubuntu1804/domain/04.png){:.small }

스크립트 작성에 앞 서 스크립트 실행에 필요한 툴을 다운로드 한다.

```
$ sudo apt-get install curl jq
```

홈 서버에서 스크립트 파일을 작성한다.

```
$ sudo vi /usr/local/etc/ddns-cloudflare.sh
```

아래 내용을 추가한다. 5~8 라인은 자신에 맞게 설정한다. 
- **EMAIL** : 로그인 이메일 주소
- **KEY** : Global API Key
- **DOMAIN** : 도메인
- **A_RECORD_LIST** : 갱신할 A 레코드 리스트. `,`로 구분한다. DNS 레코드에 없을 경우 자동으로 생성된다. 

<script src="https://gist.github.com/jkpark/c2cfbb84728feb05c20c5d3950e10f20.js"></script>

편집 후 실행권한을 추가한다.

```
$ sudo chmod 700 /usr/local/etc/ddns-cloudflare.sh
```

주기적으로 IP주소를 확인하고 업데이트 되도록 crontab에 등록한다.

```
$ sudo vi /etc/crontab
```

아래 내용을 추가한다.

```
# 2019-03-08 | jkpark | added below 1 line.
04 *    * * *   root    /usr/local/etc/ddns-cloudflare.sh
```

위 내용은 매시 4분마다 `/usr/local/etc/ddns-cloudflare.sh`가 실행시킨다는 내용이다.

# 도메인 인증서 발급

도메인에 대한 보안 인증서를 발급하여 `HTTPS`, `VPN` 등을 이용할 때 도메인을 신뢰하도록 해야한다. 신뢰되지 않은 도메인에 접속할 경우 아래와 같이 경고 문구가 뜬다.

![Invalid authority](/images/ubuntu1804/domain/05.png){:.small }

인증서 발급은 [Let's Encrypt](https://letsencrypt.org/)에서 무료로 발급할 수 있다. Let's Encrypt 에서 발급하는 인증서의 유효기간은 90일이다. 인증서 발급 방식도 여러가지가 있는데 `CloudFlare`에서 `Let's Encrypt`의 인증서 봇인 certbot에 대한 플러그인을 제공하기 때문에 쉽게 발급받고 갱신할 수 있다.

## CloudFlare API credentials
인증서 발급에 쓰일 `CloudFlare`의 계정 정보를 파일로 저장하여 발급 및 갱신 시 자동으로 참조할 수 있도록 한다.

```
$ sudo mkdir /root/.secrets
$ sudo chmod 700 /root/.secrets
$ sudo touch /root/.secrets/cloudflare.ini
$ sudo chmod 400 /root/.secrets/cloudflare.ini
```

`/root/.secrets/cloudflare.ini` 파일을 연다.

```
$ sudo vi /root/.secrets/cloudflare.ini
```

`CloudFlare`의 계정 정보를 파일에 입력한다.

```
dns_cloudflare_email = "account@email.address"
dns_cloudflare_api_key = "df8_____your_global_api_key_____d5526"
```

`:wq!` 로 저장 후 편집을 종료한다. `!`는 강제로 저장한다는 의미이다.

## Certbot과 CloudFlare DNS 인증 플러그인 설치

```
$ sudo apt-get update
$ sudo apt-get install certbot
```

`certbot`의 버전을 확인한다. 0.22.0 이상이어야 한다.

```
$ certbot --version
certbot 0.23.0
```

플러그인 설치 

```
$ sudo apt-get install python3-certbot-dns-cloudflare
```

## 인증서 발급

```
$ sudo certbot certonly \
--dns-cloudflare \
--dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
--dns-cloudflare-propagation-seconds 60 \
--preferred-challenges dns-01 \
--server https://acme-v02.api.letsencrypt.org/directory \
-d exsample.com,*.exsample.com
```

`/root/.secrets/cloudflare.ini` 파일을 참조하여 인증서를 발급받는다. `-d exsample.com,*.exsample.com`에 인증서를 발급받을 도메인을 입력하면 되는데, `*`은 와일드카드로, 모든 서브 도메인을 뜻한다. 

명령어를 실행하면 몇 가지 정보를 입력해야 한다.

```
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): your-email@email.address

-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: a

-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: y
```

60초간 DNS 레코드 변경사항이 적용되기를 기다리면 인증서가 발급된다.

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/exsample.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/exsample.com/privkey.pem
   Your cert will expire on 2019-06-06. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

내 인증서는 2019-06-06 에 만료된다고 한다. 만료되기 전 인증서가 자동갱신 해야 하는데 [certbot은 기본적으로 인증서를 자동갱신되도록 crontab에 등록되어 있다.](https://certbot.eff.org/docs/using.html#automated-renewals) 내 시스템의 경우 cerbot가 설치되면서 `/etc/cron.d/certbot` 파일이 생성되었다. 

> 갱신 작업이 제대로 작동하는지 확인하고 싶다면 `sudo certbot renew --dry-run` 명령어로 갱신 시뮬레이션을 할 수 있다.


# 끝
