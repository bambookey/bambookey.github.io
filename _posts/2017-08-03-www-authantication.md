---
layout: post
title:  WWW-Authorization验证
category: tech
---

定时任务有个比较简陋的管理后台，所以增加了个登录界面，使用了WWW-Authorization验证。


```Java
if (!expectedPassword.equals(sessionPassword) || !expectedUserName.equals(sessionUserName)) {
    if (authValue != null) {
        BASE64Decoder decoder = new BASE64Decoder();
        String[] values = new String(decoder.decodeBuffer(authValue.split(" ")[1])).split(":"); //通过解析后的用户名和密码格式例如 123:123
        if (values.length == 2) {
            username = values[0];
            password = values[1];
        }
    }
    if (expectedPassword.equals(password) && expectedUserName.equals(username)) {
        session.setAttribute("username", username);
        session.setAttribute("password", password);
    } else {
        httpresp.setStatus(401);
        httpresp.setHeader("WWW-Authenticate", "Basic realm=\"QiyeBackend\"");
        httpresp.setContentType("text/html; charset=UTF-8");
        httpresp.getWriter().print("Login Failed");
        return;
    }
}
```
