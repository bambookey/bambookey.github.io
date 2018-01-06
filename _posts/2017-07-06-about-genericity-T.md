---
layout: post
title:  关于泛型
category: tech
---

1. 泛型类

2. 泛型方法

```
public <T, A, B> T getObject(Class<T> classT, Class<A> classA, Class<B> classB) throws IllegalAccessException, InstantiationException {
    return classT.newInstance();
}
```

- 一个泛型方法首先要对出现的泛型类进行声明，即开头的<T,A,B>；
- 经过声明的泛型类即可以出现在 **参数** 或 **返回** 类型中了
- 参数指明了泛型的具体类型，要用 **Class<T>** 表示。
- Class<T> classT: 表示一个泛型类的对象classT，这个对象可以代表一个类，而Class<T>则表示这个类的类，因此这里就不能用Class T这样来表示了
