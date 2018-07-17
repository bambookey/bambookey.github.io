---
layout: post
title: 升级全程HTTPS过程中的记录
category: tech
---

最近公司在将服务升级成全HTTPS访问

之前邮件服务依赖好多子服务，这些服务多数是HTTP协议的，这次需要把他们统一修改成HTTPS

先来说简单的：

最简单的就是修改一下URL,这些URL包含静态资源，跳转链接等；

在这里也了解到了一个trick,就是在定位资源的路径中，不要去自己指定获取静态资源究竟要使用哪种协议。

比如像下面这样设置
```
<img src="//xxx/logo.png">
```
img 中的url是以双斜杠“//”开头的，这种写法有特殊的用途，它会判断当前的页面协议是http 还是 https 来决定请求 url 的协议

说完简单的再说一个棘手的，有一个服务没有装nginx，这台机器上的服务不能使用HTTP默认的80端口进行转发，
所有的服务都是hots:8080这种方式写死的，而这台机器上也不能安装证书，因此在升级全域HTTPS的过程中需要对这台机器及相关服务进行处理，
这里大致记录一下处理的流程。

1. 开放这台机器的80端口：可以选择安装Nginx，操作系统级别的80端口直接转发，我在解决的过程中尝试配置tomcat server.xml中的service，
但没有奏效，因为tomcat配置端口数不能低于1024，防止冲突

2. 开通80端口之后，我们就可以通过其他机器对这台机器进行转发，选一台可以安装证书的入口机，将之前部署相关服务中的所有含有Host:8080的
hardcode全部使用host直接进行访问，之前采用指定8080访问的方式真的是难以忍受；

3. 修改DNS解析。将原来访问8080机器的域名重新指定到入口机上，再通过入口机进行转发，转发过程中，直接转发到新机器的8080端口即可。
这里学到了dig命令，可以获取指定域名的入口IP。这里重点需要考虑的是，DNS解析需要时间，而这个时间是不可控的，所以我们要保证在DNS重新指向的期间，
服务的可用性，因此这也是第一步一定要保证原服务80端口和8080端口均可用的原因所在。另外，最后修改DNS解析的时候使用了A解析，其实也是可以使用CNAME解析的；
之前看过DNS解析，不过今天突然都忘记了，感觉用的少，可以和dig命令一起看。

4. 在修改过程中，由于以后不再支持8080端口访问服务，而是统一使用Https的方式经过入口机进行访问， 所以也要对调用该服务的其他用户和系统进行
通知和排查；另外，在排查过程中，将内部调用的情景换成内网域名进行调用。

经过一天的修改、排查、重新上线，这个服务可以支持HTTPS了，活比较杂，不过能学到很多东西，真正解决实际问题的时候是用得上的。