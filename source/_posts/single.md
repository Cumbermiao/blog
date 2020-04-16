---
title: '单例模式'
date: 2019-01-29 20:20:59
tags: [设计模式]
published: false
hideInList: false
feature: 
isTop: false
---

## 单例模式 Singleton
> 单例模式（Singleton），也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。

### 实现思路
一个类能返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法，通常使用getInstance这个名 称）；当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用；同时我们 还将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例。 

下面通过一个简单的登录表单的例子来解释实现。
```js
class LoginForm{
  constructor(){
    this.state='hide'
  }
}

//使用闭包，判断instance实例是否存在
LoginForm.getInstance = (function(){
  let instance;

  return function(){
    if(!instance){
      instance = new LoginForm()
    }
    return instance
  }
})()
```

### 优点
1. 在单例模式中，活动的单例只有一个实例，节约系统资源，当 需要频繁创建和销毁的对象时单例模式无疑可以提高系统的性能。

2. 避免对共享资源的多重占用，就如登录框，采用单例模式就不需要担心会出现多个登录框。