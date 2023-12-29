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

블로그를 처음 시작한다면 `jekyll`을 추천한다. github 에서 공식적으로 Jekyll을 추천하며 자료도 많아서 쉽게 시작할 수 있다. 나도 markdown형식 블로그로 갔을 때 jekyll을 선택했었고 jekyll로 운용한지 벌서 3년이 되었다. 

그런데 jekyll을 쓰면서 불편한 점이 하나 있었다.

page tree 구조를 만들기 어렵다는 점.

홈 서버 관련 글들을 트리 형식으로 관리하고 싶었는데 jekyll에서는 그 방법이 복잡하게 되어있었다. 그래서 gitbook 에서 홈 서버 관련 블로그르 운영 하였지만 gitbook에서는 한글 입력 버그가 있고 테마도 수정하기 어려웠기 때문에 gitbook도 접게 되었다. 그동안 jekyll에 대해 충분히 파악했고 또 다시 떠날 때가 되었다는 느낌을 받아 새로운 website publisher를 찾게 되었다.

## 블로그 운영 연혁
|년도| 블로그 운용 방식 | 이유 |
|:---:|---|---|
|2009| Personal Web Server + Zeroboard | 블로그 운용 시작 |
|2010| Naver blog | 전기세, 보안문제 등의 이유로 만만한 Naver Blog로 이전|
|2011| Blogger | Naver블로그 글은 구글 검색이 안되서 이전|
|2017| github page + jekyll | Markdown와 Static Site에 흥미를 느껴 이전 |
|2019| github page + jekyll, gitbook | jekyll은 page tree가 안되서 gitbook 추가 운영 |
|2021| github page + Hugo | gitbook의 한글 문제와, jekyll의 page tree 미지원, Go lang에 흥미를 느껴 이전 |


# Why Hugo

나는 블로그를 static으로 운영한다. 글 작성은 나만 하기도 하고 코멘트는 ecosystem으로 잘 작동하기 때문에 CMS는 불필요하다. 

static site generator도 jekyll, hugo, gatsby, hexo 등 많이 있지만 hugo는 충분히 많은 자료가 나오고 최근 Go lang도 공부하고 있기 때문에 흥미를 느꼈다.

React가 좋은 gatsby도 고려했지만 JSX나 NPM, React 등 배워야할 것들이 너무 많았기 때문에 간단한 hugo를 선택했다.


# Getting Started

Hugo를 쓰기 위해선 다음과 같은 것들을 알아야 한다.

- HTML
- CSS
- Hugo Template
- Markdown

Hugo Template에 정의된 형식으로 build하면 site가 만들어진다.

기본적인 사용 방법은 [공식 홈페이지](https://gohugo.io/getting-started/quick-start/) 에서 차근차근 알아갔다.

# Theme

hugo로 오면서 테마를 만들었다. 언젠가 [hugo themes](https://themes.gohugo.io/)에 등록할 예정이다.

내 테마를 보고 싶다면 아래 링크를 참고한다.

https://jkpark.github.io/hugo-theme-jkpark/docs/getting-started/installation/