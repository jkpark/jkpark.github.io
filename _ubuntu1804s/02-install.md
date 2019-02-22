---
layout: post
title: 2장. Ubuntu Server 18.04 LTS 설치
category: ubuntu1804
---

# Overview

시스템에 우분투를 설치할 것이다. 홈 서버를 구축하고 서브PC용으로 사용하기 위해 그래픽-모드가 설치되어야 한다. 일반 사용자라면 Ubuntu Desktop 을 설치하여 이런 부분까지 신경 쓸 필요가 없지만 Ubuntu Desktop로 설치할 경우 문서작업 툴, 브라우저, 미디어플레이어 등 사용하지 않는 기본 프로그램들이 포함되어 있다. 반면 Ubuntu Server는 네트워크과 서비스를 위한 버전으로 텍스트-모드로 설치되고 간단한 파일 서버 역활만 할 수 있는 가벼운 상태로 설치된다. 그래픽-모드로 넘어가기 위한 과정은 기본 시스템 설정을 마친 후 5장에서 다룰 것이다.

설치 과정 중 설정은 꼭 필요한 요소만 할 것이다. 설정들이 어느 위치를 참조하고 어떻게 동작하는지 정확하게 알고 가는 것 또한 목표로 하기 때문에 지역, 언어, 네트워크 등 가능한 모든 설정은 설치 후 수동으로 설정할 것이다.

# 설치 이미지 다운로드

Ubuntu Server 18.04 는 두 가지 인스톨러을 지원한다.
- Subiquity Installer (Default, ubuntu-18.04-live-server-amd64.iso)
- Debian Installer (Alternative, ubuntu-18.04-server-amd64.iso)

![Subiquity installer version](/images/ubuntu1804/00-subiquity.png){:.small }
*Ubuntu 18.04 LTS Server Live (Subiquity)*

![Debian installer version](/images/ubuntu1804/00-debian.png){:.small }
*Ubuntu 18.04 LTS Server (Debian Installer)*

Subiquity는 Ubuntu 18.04부터 새로운 인스톨러로 등장했고 UI가 조금 더 현대적이다. 두 installer의 가장 큰 차이는 `cloud-init` 설치 여부이다. Subiquity installer로 설치할 경우 cloud-init이 설치되는데, cloud-init는 우분투 설치 후 계정, 설정, 패키지 등을 cloud를 통해 초기화하는 기능을 담고 있다. 이 기능은 가령, 서버를 100대 설치할 경우 훨씬 효율적이고 빠르게 설정을 하도록 도와준다. 하지만 홈 서버의 경우 이러한 기능은 불필요하다. 또한 cloud-init는 네트워크 설정에도 관련되어 있기 때문에 네트워크 설정 방법도 달라진다. 그렇기 때문에 Debian Installer로 우분투를 설치할 것이다.

Debian Installer가 포함된 [ubuntu-18.04.2-server-amd64.iso](http://cdimage.ubuntu.com/releases/18.04.2/release/ubuntu-18.04.2-server-amd64.iso) 을 다운로드 한다. 혹은 [우분투 다운로드 페이지](http://cdimage.ubuntu.com/releases/18.04.2/release/) 에서 install image를 다운로드한다


# USB에 설치 이미지 굽기

[참고](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-ubuntu#0)

# 과정

화면 캡처를 위해 설치는 가상머신에 진행하였다. 실제와 동일한 환경을 구성하기 위해 두개의 디스크(20GB, 40GB)를 생성하였다. 

USB를 꽂고 전원을 키면 아래와 우분투 서버를 설치하는 화면이 나온다.
![install ubuntu server](/images/ubuntu1804/00.png)

<br>

![select a language](/images/ubuntu1804/01.png)
언어는 영어를 선택한다. 한글를 선택할 경우 설치도 한글로 진행된다. 한글로 설치할 경우 에러 메세지 또한 한글로 나오는데 구글링 하는데 있어서 오히려 불편하게 느껴진다. 요즘엔 안그러겠지만, 14년도엔 해도 몇몇 툴에서 한글로 설정된 디렉토리를 인식하지 못하는 에러도 있었다.

![select your location](/images/ubuntu1804/02.png)
지역 설정은 미국으로 한다. 우분투 소프트웨어들은 각종 패키지들을 모아놓은 아카이브를 통해 다운받을 수 있는데 어느 지역을 선택하느냐에 따라 아카이브 호스트가 결정된다. 우분투의 메인 아카이브의 위치는 http://archive.ubuntu.com/ubuntu 이다. 미국을 선택하면 공식 미러 아카이브 http://us.archive.ubuntu.com/ubuntu 를 사용하게 되고, 한국이나 다른 나라를 선택하게 되면 해당 지역에 맞는 미러 아카이브가 선택된다. 한국에서 한국에 있는 미러 아카이브를 선택하면 속도가 빠르다. 하지만 이 단계에서는 원하는 아카이브를 선택할 수 없기 때문에 미국을 선택하고 설치가 끝난 후 카카오에서 제공하는 미러 아카이브로 변경할 것이다.

>우분투의 아카이브 미러 리스트는 https://launchpad.net/ubuntu/+archivemirrors 에서 볼 수 있다. 

![configure the keyboard 1](/images/ubuntu1804/03.png)
키보드 레이아웃을 자동으로 설정할 것인지 묻는 화면이다. No를 선택하여 수동으로 설정한다.

![configure the keyboard 2](/images/ubuntu1804/04.png)
키보드 레이아웃은 국가에 따라 선택할 수 있는 레이아웃이 여러개로 나뉜다. 한글로 선택할 경우 101/104키보드를 선택할 수 있고 기본 한글입력기로 ibus가 설치된다. ibus는 한글 입력 시 백스페이스가 제대로 안되는 버그가 있다. 5장에서 그래픽-모드를 설치와 함께 `nimf` 한글입력기를 설치할 것이다.

![configure the keyboard 3](/images/ubuntu1804/05.png)
English (US) 를 선택한다.

![enter the hostname](/images/ubuntu1804/06.png)
호스트 네임은 기본적인 이 서버의 이름이고 다른 네트워크에서 연결 시 보여진다. 나는 서버의 이름을 'cactus'로 정했다.

![enter full name for user](/images/ubuntu1804/07.png)
사용자의 실제 이름을 입력하는 창이다. 여러 프로그램에서 친절하게 이 이름으로 불러준다. 나는 `JK Park`으로 정했다.

![enter full username](/images/ubuntu1804/08.png)
사용자 계정명을 입력한다. 로그인 시 이 계정명으로 로그인하게 된다. 나는 `jkpark`으로 정했다.

![enter password for user](/images/ubuntu1804/09.png)
비밀번호를 입력한다.

![re-enter password for user](/images/ubuntu1804/10.png)
비밀번호를 한 번 더 입력한다.

![asking if user weak passwprd](/images/ubuntu1804/11.png)
경우에 따라 취약한 비밀번호를 입력할 경우 보안이 취약하다는 안내 문구가 나온다. Yes를 눌러 넘어간다.

![configure the clock](/images/ubuntu1804/12.png)
네트워크가 연결되어 있다면 자동으로 타임존을 발견하지만 프록시를 사용하거나 네트워크 연결에 문제가 있다면 위와 같이 수동으로 타임존을 선택해야 한다. 설치 후 타임존을 설정할 것이므로 아무거나 선택한다.

![partition disk 1](/images/ubuntu1804/13.png)
이제 컴퓨터에 장착된 디스크들의 파티션을 설정할 것이다. `Manual`을 눌러 수동으로 설정한다.

파티션을 나눌 때 해당 파티션의 용도를 잘 고려해야한다. 파티션을 알맞게 나누면 파일 정리가 쉬워지고 규모가 큰 서버의 경우 /var, /tmp, /home, /usr 등 용도에 따라 파티션을 나누어 파일시스템에 충돌이 발생할 경우를 대비한다. 하지만 우리가 구성하는 작은 규모의 홈 서버의 경우 설치하는 프로그램 수, 쌓이는 로그의 수, 유저의 수 등을 고려했을 때 1GB도 안되게 여러 파티션으로 나누는 것은 오히려 비효율적일 수 있다. 또한 한 번 정한 파티션의 크기를 줄이고 늘리는 작업은 번거로울 수 있다. 

한 가지 더 언급해야 하는 것이 있다. 리눅스에서는 `swap`이라는 공간을 있다. 리눅스에서는 메모리가 부족할 경우 오랫동안 사용하지 않은 소프트웨어의 메모리를 swap 공간에 잠시 보관시켜 메모리 공간을 확보한다. swap을 설정하지 않았거나 swap 공간 마저 부족하다면 OOM-killer에 의해 어떤 소프트웨어가 죽을지 모른다.

그러나, 시대가 발전하면서 가정에서도 적당한 가격으로 16GB, 32GB RAM을 구성할 수 있게 되어 swap 공간의 필요성에 대해 논쟁을 하고 있다. 하지만 사용하는 목적에 따라 swap 공간을 확보하는 것이 바람직하다. 내 시스템은 8GB RAM을 가지고 있다. 그래픽-모드로 설치를 마친 우분투가 차지하는 메모리는 3GB가 넘기 때문에 홈 서버와 서브PC용으로 사용하기에는 부족할 것이다. 나는 8GB의 swap 공간을 확보할 것이다.

![partition disk 2](/images/ubuntu1804/14.png)
처음 언급했던 것처럼 가상머신에서 설치를 진행하기 때문에 `Virtual Disk 1 (vda)`과 `Virtual Disk 2 (vdb)`가 보인다. 실제 시스템에서 `Virtual Disk 1`는 128GB SSD다. `Virtual Disk 1`를 선택한다.

>실제 시스템에서 `Virtual Disk 2`는 3TB HDD다. HDD는 백업 및 복원이 가능한 `btrfs` 파일시스템으로 설정할 것이다. 설정이 다소 복잡하므로 3장을 통해 수동으로 설정할 것이다.

>__디스크에 부트로더가 없다면 설치 마지막 단계에서 GRUB 설치여부 물어본다. 부트로더는 보통 OS가 설치되는 디스크에 설치하므로 여기서 우분투 서버가 설치되는 디스크인 *vda* 를 기억하길 바란다.__


![partition disk 3](/images/ubuntu1804/15.png)
새로운 파티션 테이블을 만든다.

![partition disk 4](/images/ubuntu1804/16.png)
화면에 출력되는 `21.5GB FREE SPACE` 공간은 실제 시스템에선 128GB의 FREE SPACE이다. 이 공간을 2개의 파티션으로 나눌 것이다. 하나는 우분투 서버가 설치될 파티션이고, 다른 하나는 swap 공간이다. 

> 9년동안 리눅스를 사용하면서도 파티션 나누는 것이 제일 고민스럽다. 파티션을 나누는 것에 정답이 있는 것도 아니고 추 후 필요에 따라 파티션을 쪼개고 크기도 늘리고 줄일 수 있으니 사용 목적에 따라 알맞게 나누면 될 것 같다.

![partition disk 5](/images/ubuntu1804/17.png)
새로운 파티션을 만든다.

![partition disk 6](/images/ubuntu1804/18.png)
swap 공간으로 사용할 크기만큼 제외하고 나머지 크기를 입력한다. 실제 시스템에선 swap으로 사용할 `8GB`를 제외하고 `120GB`를 입력하였다.

![partition disk 7](/images/ubuntu1804/19.png)
Primary를 선택한다.

>디스크는 보통 4개의 primary파티션으로 나눌 수 있는데 마지막 하나를 확장 시켜 더 많은 파티션을 만들 수 있도록 했다. 그래서 3개의 primary 파티션과 여러개의 logical 파티션으로 구성할 수 있다. 또, 디스크 트랙은 0부터 시작하기 때문에 Primary 0을 선택하는것이 Logical을 선택하는 것보다 논리적으로 약간이나마 빠를 것이다. 하지만 요즘 컴퓨팅 속도를 생각한다면 현실적인 이득은 없을 것이다.

![partition disk 8](/images/ubuntu1804/20.png)
Beginning 을 선택한다.

![partition disk 10](/images/ubuntu1804/21.png)
Use as 는 `Ext4`를 선택한다.<br>
Mount Point는 `/`로 한다.

![partition disk 11](/images/ubuntu1804/22.png)
다음, 나머지 공간은 swap 영역으로 지정할 것이다. 실제 시스템은 8GB이다.

![partition disk 12](/images/ubuntu1804/23.png)

![partition disk 13](/images/ubuntu1804/24.png)
실제 시스템은 8GB이다.

![partition disk 14](/images/ubuntu1804/25.png)
logical을 선택했다.

![partition disk 15](/images/ubuntu1804/26.png)
`Use as:`을 선택하여 아래 그림과 같이 `swap area`로 바꿔준다.

![partition disk 16](/images/ubuntu1804/27.png)

![partition disk 17](/images/ubuntu1804/28.png)
설정을 마친다.

![partition disk 18](/images/ubuntu1804/29.png)
파티션 설정을 마친다.

![partition disk 19](/images/ubuntu1804/30.png)
Yes를 눌러 파티션 설정을 저장한다.


![Started installation](/images/ubuntu1804/31.png)
파티션 설정이 끝나면 우분투 서버 설치가 진행된다. 진행되는 도중에도 몇가지 설정을 묻는다.

![proxy](/images/ubuntu1804/32.png)
[프록시](https://en.wikipedia.org/wiki/Proxy_server) 설정 창이다. 프록시 서버를 사용 중이라면 프록시 주소를 입력한다. 일반 가정의 경우 프록시는 없을 것이다.

![configuring tasksel](/images/ubuntu1804/33.png)
주기적으로 아카이브를 통해 업데이트 받을 수 있다. 설치 후 언제든지 설정가능하므로 원하는 것을 고른다. 나는 수동 업데이트를 원하므로 `No automatic updates`를 선택하였다.

![software selection](/images/ubuntu1804/34.png)
추가 소프트웨어를 선택할 수 있다. 대부분 홈 서버에서 필요하지 않는 소프트웨어들이다. `Samba`와 `OpenSSH`는 추 후 설치할 것이다.

![still on installation](/images/ubuntu1804/35.png)
우분투 서버 설치가 아직 진행되고 있다.

![install the GRUB boot loader on a hard disk](/images/ubuntu1804/36.png)
디스크에 boot loader가 없다면 GRUB 부트로더 설치 여부를 묻는다. 부트로더가 없다면 부팅되지 않기 때문에 꼭 설치해야 한다.

![install the GRUB boot loader on a hard disk 2](/images/ubuntu1804/37.png)
디스크의 종류에 따라 약간씩 이름이 다를 수 있다. 디스크 이름은 파티션 설정 단계에서 확인하였다.  나는 `/dev/vda`를 선택하였다.

![finishing the installation](/images/ubuntu1804/38.png)
설치가 마무리되어간다.

![finished the installation](/images/ubuntu1804/39.png)
USB를 제거하고 `continue`를 눌르면 재부팅이 되면서 아래와 같이 로그인 창이 뜰 것이다.

![first boot and logon screen](/images/ubuntu1804/40.png)

계정과 비밀번호를 입력하여 로그인한다.

![login succeed](/images/ubuntu1804/41.png)

커서가 위치한 `jkpark@cactus:~$` 을 살펴보면 다음과 같다.
- `jkpark` : 사용자명
- `cactus` : 호스트 명. 시스템의 이름이다.
- `~` : 디렉토리 위치. 자신이 위치한 디렉토리를 표시한다. `~`은 사용자의 홈 디렉토리이다.
- `$` : 로그인한 사용자가 `root`권한을 갔는다면 `#`이 표시되고 아니라면 `$`이 표시된다.

리눅스의 모든 시스템에 대한 권한을 가진 사용자는 `root`이다. 우분투에서 기본적으로 `root` 계정으로 로그인이 불가능하게 설정되어 있다. root권한이 필요한 파일에 접근하기 위해선 일반 사용자가 root 권한을 빌려야 하는데 이러한 root 권한을 빌리 수 있는 사용자 그룹이 `sudoers` 그룹이다. 우분투 서버를 설치할 때 생성했던 사용자 `jkpark`는 일반 사용자이지만 `sudoers` 그룹에 속해있다. root권한이 필요한 작업 수행 시 명령어 맨 앞에 `sudo`를 입력하여 수행할 수 있다.

`whoami` 명령어는 명령어를 수행하는 사용자가 누구인지 출력해 주는 명령어이다. 

```
$ whoami
jkpark
$ sudo whoami
root
```

내 글에서는 위와 같이 명령어(입력)와 출력을 구분하기 위해 `$` 을 맨 앞에 표시하였다. `$ whoami`는 'whoami' 명령어를 수행한다는 의미이고 `$`이 붙지 않는다면 출력 결과물이라는 의미이다. 보이는 것과 같이 'sudo'를 맨 앞에 붙여 명령어를 실행하면 root권한으로 명령어를 수행하기 때문에 `root`가 출력되는 것을 볼 수 있다.


# 인터넷 연결 설정

우분투 서버가 설치되었다면 가장 먼저 IP 설정을 해주어야 한다. 대부분의 경우 IP주소는 공유기의 DHCP 기능을 통해 동적으로 할당되고 언제든지 바뀔 가능성이 있기 때문에 IP주소가 바뀌지 않도록 설정하도록 한다. 고정 IP를 사용하기 위해선 공유기에서 시스템에게 고정 IP를 할당해주거나, 시스템에서 자신이 사용할 IP를 공유기에게 요청해야 한다. 첫번째 방법을 사용한다면 IP충돌이 발생하지 않는다. 두번째 방법의 경우 이미 해당 IP주소를 누군가 사용중이라면 IP충돌이 발생할 수 있다. 그럴 경우 공유기에 접속해 IP를 중복으로 사용중인 기기의 연결을 끊어주어야 한다. 요즘 공유기는 한번 할당했던 IP주소는 MAC주소를 기반으로 관리되기 때문에 다른 기기에게 동일한 IP주소를 할당해주는 경우는 드물기 때문에 시스템에서 자신이 사용할 IP주소를 공유기에게 요청하는 방법만 설정해도 무방하다.

시스템에 인터넷 선이 잘 연결되어있는지 확인 후 아래 명령어를 입력하여 내 네트워크의 상태를 확인한다.
```
$ ip addr
```

현재 시스템의 네트워크 인터페이스가 출력될 것이다.
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:93:eb:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.74/24 brd 192.168.122.255 scope global dynamic ens3
       valid_lft 2112sec preferred_lft 2112sec
    inet6 fe80::5054:ff:fe93:eb53/64 scope link 
       valid_lft forever preferred_lft forever
```

`ens3`이 외부 인터넷 망과 연결되는 인터페이스다. 내 가상머신의 IP주소는 `192.168.122.74`로 할당되었다. 공유기를 사용 중이라면 일반적으로 IP 주소가 `192.168`로 시작한다. IP주소 뒤에 붙은 `/24`는 네트워크 클래스를 의미한다. 가정에서 공유기를 사용하는 환경이라면 일반적으로 C클래스인 24를 사용한다.

## 1. IP 설정

우분투 18.04 에서는 `netplan`을 통해 네트워크를 설정한다. 고정 IP 주소 설정을 위해 `/etc/netplan` 디렉토리에 있는 `01-netcfg.yaml` 파일을 수정해야 한다. (Subiquity 인스톨러로 우분투 서버를 설치하였다면 `50-cloud-init.yaml` 파일을 수정한다.)
```
$ sudo vi /etc/netplan/01-netcfg.yaml
```

`01-netcfg.yaml` 파일을 열어 보면 아래와 같이 DHCP를 통해 네트워크를 설정하도록 되어있다.
```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: yes
```

`ens3` 네트워크 인터페이스를 고정 IP으로 설정하기 위해 아래 내용을 입력한다.

- *addresses: [192.168.122.3/24]*  : 192.168.122.3 은 자신이 사용할 IP와 네트워크 클래스
- *gateway4: 192.168.122.1* : 게이트웨이 주소
- *nameservers: addresses: [192.168.122.1]* : DNS server. 여러개라면 `,`로 구분한다.

결과는 아래와 같아야 한다. 공백(들여쓰기) 또한 정확하게 입력해야 한다.
```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp4s0:
      dhcp4: no
      addresses: [192.168.122.3/24]
      gateway4: 192.168.122.1
      nameservers:
        addresses: [192.168.122.1]
```

파일 수정을 마쳤다면 `:wq`를 입력하여 저장 후 빠져 나온다. `:`은 명령어모드이고 `w`는 저장, `q`는 빠져 나오는 것을 의미한다.

아래 명령어를 통해 수정사항을 적용한다.
```
$ sudo netplan apply
```

설정에 문제가 없다면 `ip addr` 명령어를 입력하였을 때 IP주소가 `192.168.122.3`으로 바뀐 것을 볼 수 있다.

```
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:93:eb:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.3/24 brd 192.168.122.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe93:eb53/64 scope link 
       valid_lft forever preferred_lft forever
```

## 2. Proxy 설정

프록시를 사용한다면 프록시 설정도 한다. 프록시를 사용하지 않는다면 넘어가도록 한다.

### 환경변수에 프록시 주소 추가

프록시 주소는 `/etc/environment` 파일에 추가한다. `/etc/environment`은 모든 사용자가 로그인 시 적용되는 파일이다.

`/etc/environment` 파일을 연다.
```
$ sudo vi /etc/environment
```

아래 내용을 추가한다.
```
HTTP_PROXY=http://xx.xx.xx.xx:xx
HTTPS_PROXY=http://xx.xx.xx.xx:xx
http_proxy=http://xx.xx.xx.xx:xx
https_proxy=http://xx.xx.xx.xx:xx
no_proxy="localhost"
```

`:wq` 을 입력하고 엔터키를 눌러 저장 후 나온다.

### apt Proxy

우분투는 `apt` 명령어를 통해 소프트웨어을 관리하게 된다. `apt` 툴에서 외부와 통신이 가능하도록 프록시 주소를 입력한다.

`/etc/apt/apt.conf` 파일을 연다.
```
$ sudo vi /etc/apt/apt.conf
```

아래 내용을 추가한다.
```
Acquire::http::Proxy "http://xx.xx.xx.xx:xx";
Acquire::https::Proxy "http://xx.xx.xx.xx:xx";
```

## 소프트웨어 업데이트

IP와 프록시를 설정했다면 인터넷 연결이 되었을 것이다. 우분투 서버에 설치되어 있는 소프트웨어를 최신으로 업그레이드하기 위해 먼저 `apt-get update`로 패키지가 모여있는 아카이브와 동기화시킨다.
```
$ sudo apt-get update
```

그 후 우분투 서버에 설치되어 있는 소프트웨어들은 최신 버전으로 업그레이드를 한다.

```
$ sudo apt-get upgrade
```

# APT 아카이브를 카카오 아카이브로 변경

`apt`는 `/etc/apt/sources.list` 에 적혀있는 Source에서 패키지를 받아온다. 
먼저 `/etc/apt/sources.list` 파일 내용을 출력하여 Source 리스트를 확인한다.
```
$ cat /etc/apt/sources.list
# 

# deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

#deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ bionic main restricted
# deb-src http://us.archive.ubuntu.com/ubuntu/ bionic main restricted

...

```

우분투 서버 설치 과정 중 지역을 미국으로 선택했다면 아카이브 Source가 `http://us.archive.ubuntu.com/ubuntu/` 일 것이다. `apt-get upgrade`를 했을 때 느꼈겠지만 미국에 있는 아카이브를 통해 소프트웨어를 다운받았기 때문에 속도가 다소 느렸다. 카카오 에서 제공하는 우분투 미러 아카이브로 바꿔준다.

`/etc/apt/sources.list` 파일을 백업한다.
```
$ sudo cp /etc/apt/sources.list /etc/apt/sources.list.ori
```

`sed` 명령어를 통해 `us.archive.ubuntu.com` 를 `mirror.kakao.com`로 바꿔준다.

```
$ sudo sed -i -e 's/us.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
```

`/etc/apt/sources.list` 파일을 출력하면 아래와 같이 `mirror.kakao.com` 로 바낀 것을 확인할 수 있다.
```
$ cat /etc/apt/sources.list
# 

# deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

#deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirror.kakao.com/ubuntu/ bionic main restricted
# deb-src http://mirror.kakao.com/ubuntu/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirror.kakao.com/ubuntu/ bionic-updates main restricted
# deb-src http://mirror.kakao.com/ubuntu/ bionic-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirror.kakao.com/ubuntu/ bionic universe
# deb-src http://mirror.kakao.com/ubuntu/ bionic universe
deb http://mirror.kakao.com/ubuntu/ bionic-updates universe
# deb-src http://mirror.kakao.com/ubuntu/ bionic-updates universe

...

```


`apt-get update` 시 http://mirror.kakao.com/ubuntu을 통해 업데이트 되는 것을 볼 수 있다.
```
$ sudo apt-get update]($ sudo apt-get update
Get:1 http://mirror.kakao.com/ubuntu bionic InRelease [242 kB]
Get:2 http://mirror.kakao.com/ubuntu bionic-updates InRelease [88.7 kB] 
Get:3 http://mirror.kakao.com/ubuntu bionic-backports InRelease [74.6 kB])

...

```