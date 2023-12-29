---
layout: post
title: byteToHexString
description:
category: blog
tags: [Android]
---


```java
public class Main {
 public static void main(String[] args) {
  System.out.print(byteToHexString("abcde".getBytes()));
 }

 public static String byteToHexString(byte[] b) {
  char[] digit = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F' };
  StringBuffer sb = new StringBuffer();
  for (byte i : b) {
   sb.append(digit[((i & 0xF0) >> 4)]);
   sb.append(digit[i & 0x0F] + " ");
  }
  return sb.toString();
 }
}
```