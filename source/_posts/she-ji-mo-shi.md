---
title: '设计模式'
date: 2020-03-21 15:51:03
tags: [设计模式]
published: true
hideInList: true
feature: /post-images/she-ji-mo-shi.jpg
isTop: false
---
## 分类
### 创建型
- 工厂模式
- 单例模式
- 原型模式

### 结构型
- 装饰器模式
- 适配器模式
- 代理模式
- 外观模式
- 桥接模式
- 组合模式
- 享元模式

### 行为型
- 策略模式
- 模板方法模式
- 观察者模式
- 迭代器模式
- 职责链模式
- 命令模式
- 备忘录模式
- 状态模式
- 访问者模式
- 中介者模式
- 解释器模式

## 面向对象
> 面向对象三要素： 继承、封装、多态。

### 继承
> 子类可以继承父类的属性及方法。
继承可以将公共方法抽离， 提高复用，减少冗余。

### 封装
> 数据的权限和保密。
public 类和实例都能访问、protected 子类及当前类可访问、private 只限当前类访问。
减少耦合，不该外露的不外露。
利于数据、接口的权限管理。

### 多态
> 同一接口不同实现， 需要结合 java 等语言的接口、重写、重载功能。
在 js 中具体体现在子类对父类方法的重写。
保持子类的开放性和灵活性。

### 为何使用面向对象
面向对象的实质是数据结构化。
对于计算机，结构化的才是最简单的。

## 工厂模式
- 将 new 操作单独封装
- 遇到 new 时， 就要考虑是否该使用工厂模式
```js
class jquery{
    constructor(selector){
        this.selector = selector
    }   
    html(){}
    css(){}
}
window.$ = function(selector){
    return new jquery(selector)
}
```

## 单例模式
> 一个类只有一个实例
- 依赖于 private, 在 java 中如果在外面使用 newSingleObject() 会报错。
  ```java
  public class SingleObject{
      //私有化构造函数， 外部无法使用 new 调用
      private SingleObject(){}
     // 私有化唯一实例
      private SingleObject instance = null;

      public SingleObject getInstance(){
          if(instance==null){
              instance = new SingleObject();
          }
          return instance
      }
  }
  ```
  - js 中没有 private 的概念，所以只能通过文档及使用来规范。
  ```js
  class SingleObject{
      constructor(){}
  }
  SingleObject.getInstance = (function(){
      let instance
      return function(){
          if(!instance){
              instance = new SingleObject()
          }
          return instance
      }
  })()
  ```

  ### 适配器模式
  - 旧接口格式和使用者不兼容
  - 中间加一个适配转换接口
  
  ### 装饰器功能
  - 为对象添加新功能
  - 不改变其原有的结构和功能
  可以用来装饰类和函数。
  ```js
  @decorator
class A{}

//等同于
class A{}
A = decorator(A)||A
```
