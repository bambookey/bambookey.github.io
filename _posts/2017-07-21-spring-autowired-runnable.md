---
layout: post
title:  多线程中的Spring Bean注入问题
category: tech
---

**问题描述：**

需要一个定时清理过期记录的任务，通过ExecutorService实现。在之前的项目中由于没有使用Spring，实例化工作都是通过new或工厂类来实现的；而在Spring大环境下，发现了 **多线程条件下@Autowired注入失效的问题** 其原因是由于Spring处于安全性考虑而不会对多线程类中进行注入。

**解决方法：**

在了解了问题发生原因的基础上，解决方案也就出来了，本质上是避开Spring内置的依赖注入方式。

1. 将多线程类设置为内部类，在外部类进行注入

```Java
public class RecordCleaner {
    private static final Logger LOG = LoggerFactory.getLogger(RecordCleaner.class);
    private static final String CLEAN_TIME = "01:00:00";
    private static final int PAGE_SIZE = 100; //每次删除数量
    private static ScheduledExecutorService cleanerPool = Executors.newSingleThreadScheduledExecutor();

    @Autowired
    MailDelDao mailDelDao;
    private class Worker implements Runnable {
        @Override
        public void run() {
            Date expiredDate = DateUtil.getStartDate();
            try {
                clean(expiredDate);
            } catch (Exception e) {
                LOG.error("expired record clean failed. expiredDate:{}", expiredDate, e);
            }
        }
    }
}
```

2. 将需要的Bean使用ApplicationContext重新获取

3. 自行set


**Ps:定时任务启动的小问题：**

由于定时任务启动使用的是ServletContextListener,同样无法通过依赖注入的方式获取Bean，此处采用了另一种方式：

```Java
public class WebContentListener implements ServletContextListener {

    private static final Logger LOG = LoggerFactory.getLogger(WebContentListener.class);

    RecordCleaner recordCleaner;

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        LOG.info("tomcat startup");
        WebApplicationContext webApplicationContext= WebApplicationContextUtils.getWebApplicationContext(sce.getServletContext());
        recordCleaner=(RecordCleaner) webApplicationContext.getBean("recordCleaner");
        start();
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        LOG.info("tomcat shutdown");
        stop();
    }
}

```
