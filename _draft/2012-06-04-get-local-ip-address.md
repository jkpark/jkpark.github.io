---
layout: post
title: get Local IP Address List
description:
category: blog
tags: [Android]
---


```java
private ArrayList<String> getLocalIPAddressList() {
    ArrayList<String> mAddress = new ArrayList<String>();
    try {
        for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements();) {
            NetworkInterface intf = en.nextElement();
            for (Enumeration<InetAddress> enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements();) {
                InetAddress inetAddress = enumIpAddr.nextElement();
                if (!inetAddress.isLoopbackAddress() && (inetAddress instanceof Inet4Address)) {

                        mAddress.add(inetAddress.getHostAddress().toString());
                } 
            }
        }
    } catch (SocketException e) {
        Log.e(TAG, "Fail to get IP Address : " + e);
    }

    return mAddress;
}
```
