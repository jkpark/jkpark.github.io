---
layout: post
title: edittext 특수문자 제한, 한글만 입력
description:
category: blog
tags: [Android]
---

영문 + 숫자

```java
public InputFilter filterAlphaNum = new InputFilter() {
    public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
        Pattern ps = Pattern.compile("^[a-zA-Z0-9]*$");
        if (!ps.matcher(source).matches()) {
            return "";
        }
        return null;
    }
};
```

한글

```java
public InputFilter filterKor = new InputFilter() {
    public CharSequence filter(CharSequence source, int start, int end, Spanned dest, int dstart, int dend) {
        Pattern ps = Pattern.compile("^[ㄱ-ㅣ가-힣]*$");
        if (!ps.matcher(source).matches()) {
            return "";
        }
        return null;
    }
};
```

```java
editText.setFilters(new InputFilter[]{filterAlphaNum});
```
