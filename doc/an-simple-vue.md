---
title: vue的第一份正式源码
date: 2016-11-09 
---

提交：a5e27b1174e9196dcc9dbb0becc487275ea2e84c
commit: naive implementation

这可以说是vue的第一份正式源码，已经有了基本的骨架原型。

源码主要包含三个文件: main.js、directives.js、filters.js

### 运行原理：

* 节点收集，找到根节点以及根节点之下的所有包含指令的节点.
* 解析指令节点
* 关联数据与节点
* 监测指令节点的set操作，并调用指令的更新函数

伪代码：

```js
/*根节点以及所有指令节点获取，这里指令节点的selector采用属性选择器来选择*/
root = document.getElementById(opts.id),
els  = root.querySelectorAll(selector)

/**
 * 节点处理
 */
;[].forEach.call(els, processNode)
processNode(root)

/**
 * processNode中主要做了如下三步
 */
parseDirective()
bindDirective()
bindAccessors()
```
这里需要注意一点是 scope 的概念，vue是采用数据响应式的思想，这里的数据即对应一个vue实例里的 scope(也可称它为作用域，最新版本已改为data/vueInstance.$data)。上面所说的绑定操作都是针对于这个scope来的。

如：
```js
scope = {
    hello: 'ahahah'
}
```
则在模板中声明一个`v-text='hello'`指令的时候，即实现了v-text指令与scope.hello的绑定。

如上便是当前版本vue的基本运行原理。

### 作者设计思想解读

* 通过指令的声明方式实现某一DOM片段与某一javascript对象的关联
* 数值关联，JS中的一个 String 对应于DOM中的一个或多个 textNode
* 函数关联，JS中的一个方法 对应于DOM节点的事件函数
* set监测

通过如下示例来观看作者指令语法的设计思想：

### 模板

```html
<div id="test" sd-on-click="changeMessage | .button">
    <p sd-text="msg | capitalize"></p>
    <p sd-show="something">YOYOYO</p>
    <p class="button" sd-text="msg"></p>
    <p sd-class-red="error" sd-text="hello"></p>
</div>
```
### JS
```javascript
var Seed = require('seed')
var app = Seed.create({
    id: 'test',
    // template
    scope: {
        msg: 'hello',
        hello: 'WHWHWHW',
        changeMessage: function () {
            app.scope.msg = 'hola'
        }
    }
})
```

指令语句即 DOM 节点中的一个属性，如`sd-text="msg | capitalize"`,等号前面为指令的名称，等号后面为指令的值。

受于字符串所能表达信息量的限制，作者在指令名称上采用 '-' 让指令名变成结构类型的数据，以此来增加指令的灵活性(标签的属性是不区分大小写的，所以不能采用驼峰式的命名).值对应组件作用域中的一个键名，这里通过管道符可以扩展相应的功能。

实质上来说，与普通的变量声明方式是一样的：

```js
/*javascript 变量声明*/
var text = 'ahahah';
var onClick = function changeMessage () {};

/*vue 指令声明, hello 和 changeMessage 则对应 scope 中的 hello 和 changeMessage的值*/
v-text='hello';
v-on-click='changeMessage';
```

注：组件对象数据与指令关联是一对多的关系

这样，一个微型的vue就成型了。





