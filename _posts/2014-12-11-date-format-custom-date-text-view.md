---
layout: post
title: 날짜 표현 DateFormat, Custom Date Text View
description:
category: blog
tags: [Android]
---


```java
Date today;
String output;
SimpleDateFormat formatter;

formatter = new SimpleDateFormat(pattern, currentLocale);
today = new Date();
output = formatter.format(today);
System.out.println(pattern + " " + output);
```


#### Customized Date and Time Formats

|Pattern|Output|
|--------------|
|dd.MM.yy|30.06.09|
|yyyy.MM.dd G 'at' hh:mm:ss z|2009.06.30 AD at 08:29:36 PDT|
|EEE, MMM d, ''yy|Tue, Jun 30, '09|
|h:mm a|8:29 PM|
|H:mm|8:29|
|H:mm:ss:SSS|8:28:36:249|
|K:mm a,z|8:29 AM,PDT|
|yyyy.MMMMM.dd GGG hh:mm aaa|2009.June.30 AD 08:29 AM|

<br>

#### Custom Date Text View

<script src="https://gist.github.com/jkpark/2163ac1d2bc82361ae9771bc722b5ebf.js"></script>
