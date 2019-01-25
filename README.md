Jekyll website serving [JK Park](http://jkpark.github.io)

# Checkout :

```console
$ git clone https://github.com/jkpark/jkpark.github.io.git
```

# Serve

```console
$ cd jkpark.github.io
$ jekyll serve --host=HOSTIP
```

# 홈페이지 설정

## Google Site Verification

1. https://www.google.com/webmasters/tools/home?hl=ko 에 접속
2. 속성 추가
3. 구글에서 제공하는 html 다운로드 (google20a646a1a7c513d0.html)
4. _layouts/default.html 의 head 에 자신의 태그 추가
```html
<meta name="google-site-verification" content="H73FzMq39pLvMqBQL_e5f-JlaipO9MkcGC_ce_3xlDA" />
```

## google 검색 엔진에 노출 시키기
1. _config.yml 파일의 `url` 자신의 홈페이지 주소 입력
2. 웹마스터 도구에 sitemap.xml 제출

## 구글 애널리틱스
https://analytics.google.com/analytics/web/#/ 에 접속
속성 추가
_config.yml 파일의 `google_analytics:` 에 자신의 추적 정보 입력

## Disqus

1. Disqus 설정
2. _config.yml 파일의 `disqus:` 에 자신의 shortname 입력

# Publish

```console
$ git commit -m '...'
$ git push origin master
```

# Write new post

```console
$ cd _posts
$ touch yyyy-mm-dd-slug.md
```
 - slug : 해당 포스트의 고유 URL. should use english, disits and '-' only.

 

