---
layout: post
title: NERD tree 개발버전, NERD tree git plugin 을 vundle로 설치
description:
category: blog
tags: [NERDtree, Vim, Vundle]
---

[NERD Tree](http://www.vim.org/scripts/script.php?script_id=1658) 란 아래 그림과 같이 `vim`에서 파일시스템을 탐색하고 파일을 열 수 있도록 도와주는 플러그인이다.

![](https://2.bp.blogspot.com/-pHgUOdoCZSo/WG8wSHSt_9I/AAAAAAAAAqU/ZBV2WQPN1t8zqTjsRw7IGP_HVaOpbeDbACLcB/s700/2862367534_53cd90855e_o.gif)

nerd tree git plugin은 vim 에서 git 저장소의 상태확인을 위해 쓴다.

![](https://4.bp.blogspot.com/-s4ZwVlfDR5E/WG80qtmbDVI/AAAAAAAAArE/flNXwROuSSoSoddiLlktyldBEKMxuXzMwCLcB/s700/687474703a2f2f692e696d6775722e636f6d2f6a534377476a552e6769663f31.gif)

# History

`.vimrc`를 열어 `vundle` 플러그인 삽입란에 아래 2라인을 추가한다.

```bash
Plugin 'scrooloose/nerdtree'
Plugin 'Xuyuanp/nerdtree-git-plugin'
```

![](https://2.bp.blogspot.com/-ZRfFHB6gcxU/WG81bmrj-oI/AAAAAAAAArI/2_jZCvpgsJQZgFSx4_z8Qvx2dmPjDG3BwCLcB/s700/%25EC%25BA%25A1%25EC%25B2%2598.PNG)

저장 후 `vim` 에서 `:PluginInstall` 입력

![](https://1.bp.blogspot.com/-8Ikzyvh-uSs/WG82MLbQC3I/AAAAAAAAArY/bfy3jmH5Vdc2bu5DvccK_MLN_PR4MFg0gCLcB/s700/%25EC%25BA%25A1%25EC%25B2%2598.PNG)

설치완료.
