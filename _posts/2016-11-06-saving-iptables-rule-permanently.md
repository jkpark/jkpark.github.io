---
layout: post
title: Saving Iptables Firewall Rules Permanently
description:
category: blog
tags: [iptables]
english: true
---

create `/etc/network/if-pre-up.d/iptables` file and put

```bash
#!/bin/sh
iptables-restore < /etc/iptables.rules
exit 0
```

create `/etc/network/if-post-down.d/iptables` file and put

```bash
#!/bin/sh
iptables-save -c > /etc/iptables.rules
if [ -f /etc/iptables.rules ]; then
    iptables-restore < /etc/iptables.rules
fi
exit 0
```

then 

```bash
jkpark@cactus:~$ sudo chmod +x /etc/network/if-post-down.d/iptables
jkpark@cactus:~$ sudo chmod +x /etc/network/if-pre-up.d/iptables
```