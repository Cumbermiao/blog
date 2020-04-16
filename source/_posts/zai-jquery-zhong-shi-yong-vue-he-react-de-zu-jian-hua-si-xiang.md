---
title: '在 jquery 中使用 vue 和 react 的组件化思想'
date: 2020-02-28 13:56:22
tags: []
published: true
hideInList: false
feature: /post-images/zai-jquery-zhong-shi-yong-vue-he-react-de-zu-jian-hua-si-xiang.jpg
isTop: false
---
## 起因
最近遇到一个比较奇葩的场景， 业务需求没有描述清楚导致原来逻辑全部被修改。实际需求只是增加一个类型， 在选择该类型时展示的 UI 和需要提交的表单元素都要修改。
当时在看代码时， 代码已经达到了一千多行， 对于一个页面来说还是比较多的。而且又需要快速恢复之前的逻辑并且加上现有的需求， 我一直在考虑有没有一个好的模式能够简单实现需求。
最终， 我尝试了使用类似 vue 和 react 组件的方法来实现。

## 过程
首先理一下业务需求：
实现新增编辑和查看一个表单， 在新增时有一个升舱类型3， 其对应的值为 1,2,3 ,其中 1,2 与 3 所对应的视图不一样， 具体体现在  1,2 会有座位表格信息， 3 没有座位表格信息， 1,2 的价格表格信息与 3 展现的字段不一致， 最大的区别在于 1,2 对于国内航线和国际航线可以一起提交， 3 只能选择一种航线类型。

当时的思路就是设计一个模式， 可以兼容现在的类型以及之前的类型并且可以最小改动现状的代码。当时想使用类似 vue 和 react 这种数据驱动的方式， 在 state 中定义 upgfType 变量表示 升舱类型， 根据 upgfType 不同的值， 使用不同的 UI 。

### 具体实现

#### methods & hooks 函数代理
为了拥有类似 vue 的开发体验， 我对 methods 以及 hooks 中的函数进行了 this 绑定， 绑定在 ==组件对象模型==上， 此处需要区分组件和组件模型的概念。

```js
  //所有组件对象需要调用该函数，否则 this 指向错误
  var BindThisInMethodsHooks = function(obj) {
      var methods = Object.keys(obj.methods);
      var hooks = Object.keys(obj.hooks);
      var i = 0;
      while (i < methods.length) {
          var methodName = methods[i];
          obj.methods[methodName] = obj.methods[methodName].bind(obj);
          proxyMethod(methodName, obj);
          i++;
      }
      i = 0;
      while (i < hooks.length) {
          var hookName = hooks[i];
          obj.hooks[hookName] = obj.hooks[hookName].bind(obj);
          i++;
      }

      //标记以及Bind 过 this
      Object.defineProperty(obj, "_bind", {
          configurable: false,
          get: function() {
              return true;
          }
      });
  };

  var proxyMethod = function(methodName, obj) {
      Object.defineProperty(obj, methodName, {
          get: function() {
              return obj.methods[methodName];
          }
      });
  };
  ```
以上代码中还在组件模型对象上添加了一个属性 `_bind` 用来表示该组件模型对象以及调用过 `BindThisInMethodsHooks` 绑定过 this 。 如果重复调用的话， 会报重复使用 Object.defineProperty 定义属性的错误。

#### 组件模型对象

定义的具体组件模型如下：
```js
var noop = function() {};
var emptyObj = {};

  var Base = {
      el: "", // 挂载的 dom 节点
      template: "", //html 字符串
      instance: undefined, // Component 实例
      state: emptyObj,
      props: emptyObj, //未使用
      methods: emptyObj,
      hooks: {
          beforeCreate: noop, //调用 http 初始化数据的地方
          created: noop, //根据数据渲染 template 的地方, 此处需要将 template 渲染完成
          mounted: noop, //组件已挂载， 此处用来监听事件
          beforeUpdate: noop, //组件更新前， 用于 state 改变之后， template 中结构发生变化， 卸载 mounted 中监听的事件
          updated: noop, // 组件state UI 已经更新， 用于 template 结构变化后， 重新监听事件
          beforeDestory: noop, //卸载之前, 此处用来卸载事件
          destoryed: noop //组件已卸载
      }
  };
  ```
  当时在定义 hooks 即生命周期时， 首先关注的就是 `beforeCreate` , `created`, `mounted`, `beforeDestory`。
   与 vue 不同，`beforeCreate` 生命周期我是用来获取渲染数据的， 实际场景中就是发送 http 请求， 这样使用有一个坏处： 如果你在 `beforeCreate ` 中的请求时间较长， 那么后面的 hooks 无法执行， 导致初始 UI 长时间空白。    

   `created` 周期我是用来渲染数据至 template 中的， 此周期生成展示在页面上的 html 字符串。

  ` mounted` 周期是在 html 已经插入到 el 上， 此时我们可能需要监听一些事件， 例如在业务中点击国内航线需要展示国内航线的表格， 点击国际航线需要展示国际航线的表格。以及需要使用 bootstrap-table  生成价格信息或者座位信息表格。

  `beforeDestory` 周期主要是用来卸载监听的事件， 此处比较繁琐的是你在 mounted 所有的监听事件回调都需要使用具名函数， 否则此处你无法卸载。

  以上三个周期看似够用， 但实际上在你需要更新 UI 时就相形见绌了。例如， 原本的升舱类型为 1， 此时我改变为 3， 就需要重新生成表单。
重新生成 UI 那么就要卸载之前 UI 的监听， 此处使用 `beforeUpdate` 周期来做这种事。
同样在重新生成 UI 且挂载后你需要为当前 UI 添加事件监听， 可以在 `updated`周期中处理。

在以上描述中， 我们可以看到从 `mounted` 到 `updated` 的过程中需要处理很多的事件监听以及卸载。而在 Vue 中， 由于有自己的 template 模板， 以及 @ 的事件监听指令，它可以集中处理事件的监听和卸载， 不需要  coder 自己手动卸载。 当然， Vue 也提供了自己的合成事件来减轻 coder 自己的负担。 
Vue 有了 template 模板之后， 他还可以做一件事， 拆分子组件。 在 template 解析到 options.components 一样的标签时， 它知道这是一个子组件， 从而去初始化子组件。而在我的实现中 template 是 html 字符串， 没有解析 template 的过程， 也就无法实现子组件， 没有子组件那么在根据 state 更新 UI 时， 就需要 coder 自己手动去处理事件的卸载与监听。 如果有子组件， 这些完全可以在子组件的 hooks 中实现， 当前组件的代码就会清晰很多。

#### 组件类

```js
function Component(obj) {
      if (!obj._bind) {
          BindThisInMethodsHooks(obj);
      }
      this.fiber = obj;
      this.fiber.initState = Object.assign({}, this.fiber.state);
      this.fiber.instance = this;
      var deferred = this.fiber.hooks.beforeCreate();
      var _this = this;

      if (deferred) {
          deferred
              .then(function() {
                  _this.render();
                  _this.fiber.hooks.mounted();
              })
              .fail(err => {
              console.log(err);
      });
      } else {
          _this.render();
          _this.fiber.hooks.mounted();
      }
      return this;
  }

  Component.prototype.render = function() {
      this.fiber.hooks.created();
      console.log("=========render=========");
      var $el = $(this.fiber.el);
      $el.html(this.fiber.template);
  };

  Component.prototype.rerender = function() {
      console.log("=========rerender=========");
      this.fiber.template = this.fiber.initTemplate;
      this.fiber.hooks.created();
      this.fiber.hooks.beforeUpdate();
      var $el = $(this.fiber.el);
      $el.html(this.fiber.template);
      this.fiber.hooks.updated();
  };

  Component.prototype.destory = function() {
      this.fiber.hooks.beforeDestory();
      $(this.fiber.el).html("");
      this.fiber.hooks.destoryed();
      this.fiber.state = this.fiber.initState;
      delete this.fiber.initState;
  };
  ```
  compnent 类的实现就相对简单， 其主要目的是按照顺序执行组件模型中的 hooks ， 并且提供 rerender 和 destory 方法。
  以上代码中， 用到了 jquery 中 defer 对象， 确保 `beforeCreate` 中的异步数据请求到之后再渲染 template 。
  提供 rerender 方法的主要原因是没有实现根据 state 里面的状态变更自动更新， 因为有些情况下 state 里面的数据更新之后并不需要更新页面， 所以 rerender 就留给 coder 自己去实现哪些 state 更新之后去冲渲染。例如本例中需要更加 upgfType 自动更新 UI。
 ```js
  var editModal;
      $("#exampleModal").on("shown.bs.modal", function(e) {
          console.log("relatedTarget", e.relatedTarget);
          if (e.relatedTarget === undefined) {
              formModal.state.isEdit = true;
          }
          editModal = new Component(formModal);
          var state = editModal.fiber.state;
          var upgfType = state.upgfType;
          var airlineType = state.airlineType;
          Object.defineProperties(state, {
              upgfType: {
                  get: function() {
                      return upgfType;
                  },
                  set: function(newVal) {
                      upgfType = newVal;
                      editModal.rerender();
                  }
              },
              airlineType: {
                  get: function() {
                      return airlineType;
                  },
                  set: function(newV) {
                      airlineType = newV;
                      if (state.upgfType === "3") {
                          editModal.rerender();
                      }
                  }
              }
          });
      });
```
    
## 总结

这次实践组件化的过程， 对 vue 的生命周期有了更深的体验， 当然也明白了 react 为什么要出 hooks 原因。后面如果有机会可以借助开源的 template 库自己实现子组件。
