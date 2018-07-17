---
layout: post
title: 加载配置文件获取CLASSPATH下的路径
category: tech
---

最近北京HM用户由于没有配置文件，导致北京HM用户一些mailapp功能无法正常使用，
一般这些问题都是在上线之后逐渐发现并注意修正的，个人觉得这样做很不好，实际上应该根据
项目特性，抽象出每个项目的属性，依赖等等，维护一个项目操作大盘，甚至可以统一配置文件
这样可以用机器自动化维护所谓的项目属性，使得项目不仅仅有代码执行的属性，还有一个对项目
本身的描述。还是要相信规范化的力量啊。

话说回来，这次的配置就踩了坑，配置文件加载比较蛋疼，之前的配置文件加载代码没有work，所以
自己修改了一下，顺便把可能的坑都踩了一遍，记录一下。

总结一下，实际上问题就是
Class.getResource 和 ClassLoader.getResource的区别

先随便写了一些DEMO
```
TestInHandler.class.getResource("");
// file:/D:/RD/workspace/nettydemo/target/classes/com/lxy/netty/demo/server/

TestInHandler.class.getResource(".");
// file:/D:/RD/workspace/nettydemo/target/classes/com/lxy/netty/demo/server/

TestInHandler.class.getResource("/");
// file:/D:/RD/workspace/nettydemo/target/classes/

TestInHandler.class.getResource("/xxxx");
// file:/D:/RD/workspace/nettydemo/target/classes/xxxx

TestInHandler.class.getClassLoader().getResource("");
// file:/D:/RD/workspace/nettydemo/target/classes/

Thread.currentThread().getContextClassLoader().getResource("");
// file:/D:/RD/workspace/nettydemo/target/classes/

Thread.currentThread().getContextClassLoader().getResource("");
// file:/D:/RD/workspace/nettydemo/target/classes/

Thread.currentThread().getContextClassLoader().getResource("/"); // 个人比较倾向于这种写法，万金油，适合任意
// file:/D:/RD/workspace/nettydemo/target/classes/
```

从DEMO里可以看出，Class.getResource("/") 和 ClassLoader.getResource("") 在效果上是一样的，
都是获取CLASSPATH根目录下的路径

Class.getResource("") 则是获取当前Class所在目录的路径

ClassLoader.getResource("/") 返回的则是null, 这是由于 ClassLoader 类加载器采用双亲委派制度，层级向上，
而这个采用的是 BootstrapLoader, 是C++编写，加载范围为null

通过查看源码，实际上 Class.getResource 也是通过 ClassLoader.getResource 来实现的，而 Class.getResource 可以使用"/"开头、
作为参数，则是在代码中对开头的"/"进行了处理，会去掉开头的"/"

```
/**
  * Add a package name prefix if the name is not absolute Remove leading "/"
  * if name is absolute
  */
 private String resolveName(String name) {
     if (name == null) {
         return name;
     }
     if (!name.startsWith("/")) {
         Class<?> c = this;
         while (c.isArray()) {
             c = c.getComponentType();
         }
         String baseName = c.getName();
         int index = baseName.lastIndexOf('.');
         if (index != -1) {
             name = baseName.substring(0, index).replace('.', '/')
                 +"/"+name;
         }
     } else {
         name = name.substring(1);
     }
     return name;
 }
```

至此，大致就是这样，虽然感觉自己以后不会再遇到这个坑，但还需要
