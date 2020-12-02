---
title: Jekyll to Hugo
description: Hugo로 Github Page 블로그 구축 방법 설명
draft: false
weight: 0
image: "" # relative path of /static/images folder
tags: [hugo]
category: [hugo]
type: blog
---
# Leave Jekyll

블로그를 처음 시작한다면 `jekyll`을 추천한다. github 에서 공식적으로 Jekyll을 추천하며 자료도 많아서 쉽게 시작할 수 있다. 
나도 blogger로 블로그를 운용하다가 markdown형식으로 바꿨을 때 jekyll을 선택했었고 jekyll로 운용한지 벌서 3년이 되었다. 

jekyll을 쓰면서 불편한 점은 딱 하나 있었다.

page tree 구조를 만들기 어렵다는 점.

홈 서버 구축 글들을 그룹 형태의 트리로 관리하고 싶은데 그 방법이 복잡하게 되어있었다. 그래서 홈 서버 구축 글들은 gitbook 으로 옮겼지만, gitbook에서는 한글 입력 문제도 있고 테마도 원하는 대로 수정하기 어려웠기 때문에 gitbook도 접게 되었다. 그동안 jekyll에 대해 충분히 파악했고 또 다시 떠날 때가 되었다는 느낌이 왔다.

## 블로그 운영 연혁
|년도| 블로그 운용 방식 | 이유 |
|:---:|---|---|
|2009| Personal Web Server + Zeroboard | 블로그 운용 시작 |
|2010| Naver | 전기세, 보안문제 등의 이유로 만만한 Naver로 이전|
|2011| Blogger | Naver블로그 글은 구글 검색이 안되서 이전|
|2017| github page + jekyll | Markdown와 Static Site에 흥미를 느껴 이전 |
|2020| github page + Hugo | page tree 단점과 Go lang에 흥미를 느껴 이전 |


# Why Hugo
