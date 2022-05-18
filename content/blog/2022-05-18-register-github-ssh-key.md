---
title: GitHub 로그인 없이 push 하기
description: SSH Key를 등록하여 GitHub 로그인 없이 push 하는 방법
draft: false
weight: 0
image: "" # relative path of /static/images folder
tags: [git, github]
category: blog
---



```
$ ssh-keygen -t rsa -b 4096 -C "jkpark.r@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/jkpark/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/jkpark/.ssh/id_rsa
Your public key has been saved in /home/jkpark/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:+wFoSRin+UpwTcIfc8tqXB4Ma3EFawckT34hVH+E2xQ jkpark.r@gmail.com
The key's randomart image is:
+---[RSA 4096]----+
...
hide
...
+----[SHA256]-----+
```

```
$ eval "$(ssh-agent -s)"
Agent pid 44437
```

```
$ ssh-add ~/.ssh/id_rsa
Identity added: /home/jkpark/.ssh/id_rsa (jkpark.r@gmail.com)
```

이후 github 에서 `/home/jkpark/.ssh/id_rsa.pub` 을 settings - SSH keys 에 등록