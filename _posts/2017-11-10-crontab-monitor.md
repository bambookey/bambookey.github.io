---
layout: post
title:  JVM遇到的问题
category: tech
---

清华项目由于阿里云服务器内存过小，tomcat进程经常会被杀掉，增加了一个定时脚本，在服务挂掉的时候重新启动tomcat：

```
#!/bin/sh

# 获取tomcat PPID
TomcatID=$(ps -ef |grep tomcat |grep -w 'tomcat'|grep -v 'grep'|awk '{print $2}')

# tomcat_startup
StartTomcat=/root/software/apache-tomcat-7.0.65/bin/startup.sh


#TomcatCache=/usr/apache-tomcat-5.5.23/work

# 定义要监控的页面地址
WebUrl=http://xxx.cn/

# 日志输出
TomcatMonitorLog=/root/TomcatMonitor.log
GetPageInfo=/dev/null

Monitor()
{
  echo "[info]开始监控tomcat...[$(date +'%F %H:%M:%S')]"
  if [ $TomcatID ];then
    echo "[info]tomcat进程ID为:$TomcatID."
    # 获取返回状态码
    TomcatServiceCode=$(curl -s -o $GetPageInfo -m 10 --connect-timeout 10 $WebUrl -w %{http_code})
    if [ $TomcatServiceCode -eq 200 ];then
        echo "[info]返回码为$TomcatServiceCode,tomcat启动成功,页面正常."
    else
        echo "[error]访问出错，状态码为$TomcatServiceCode"
        echo "[error]开始重启tomcat"
        kill -9 $TomcatID  # 杀掉原tomcat进程
        sleep 3
        $StartTomcat
    fi
  else
    echo "[error]进程不存在!tomcat自动重启..."
    echo "[info]$StartTomcat,请稍候......"
    $StartTomcat
  fi
  echo "------------------------------"
}
Monitor>>$TomcatMonitorLog
```

最后再crontab中加入这个脚本即可
```
*/1 * * * * /root/monitor.sh
```
