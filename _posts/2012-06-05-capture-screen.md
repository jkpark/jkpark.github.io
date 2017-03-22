---
layout: post
title: screen capture
description: 
category: blog
tags: [Android]
---


```java
public void screenshot(View view) throws Exception {

    view.setDrawingCacheEnabled(true); 
    Bitmap screenshot = Bitmap.createBitmap(view.getDrawingCache()); 
    view.setDrawingCacheEnabled(false); 

    if (screenshot == null) { 
        Log.e("screencast", "error : could not get drawing cache..."); 
        return; 
    } 

    File dir = Environment.getExternalStorageDirectory(); 
    String baseName = "screencast"; 
    String extention = ".jpg"; 
    String filename = baseName + extention; 

    try { 
        File f = new File(dir, filename); 
        f.createNewFile(); 

        OutputStream outStream = new FileOutputStream(f); 

        screenshot.compress(Bitmap.CompressFormat.JPEG, 80, outStream);
        outStream.flush(); 
        outStream.close(); 

        Log.d("screencast", "captured to " + f.getPath() + "(" + f.length() + "byte)"); 

    } catch (IOException e) { 
        e.printStackTrace(); 
    } 
}
```