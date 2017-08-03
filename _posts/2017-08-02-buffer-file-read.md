---
layout: post
title:  读输入流写文件
category: tech
---


前几天项目里需要将输入流保存到本地，上线后发现文件末尾有乱码；问题的原因是在读文件时是1k,1k的读，但是写的时候，文件未必是1k,1k的，所以末尾就会产生了多读出来的乱码，之前写了好久都没有遇到这种问题，以后应该注重细节，理解为什么这样做。

另外文件操作多可以用FileUtils等工具类，在使用的同时也应了解源码及原理，毕竟是经典代码。

最近加班加成狗。

```Java
try {
    fos = new FileOutputStream(filePath);
    byte[] b = new byte[1024];
    int l;
    while ((l = is.read(b)) != -1) {
        readTimes++;
        fos.write(b, 0, l);
    }
} catch (IOException e) {
    LOG.error("stream save failed. path:{}", filePath, e);
    throw new ApiException("STREAM.SAVE.FAILED", "path:" + filePath);
}
```
