---
title: 嵌套对象响应式支持
date: 2016-11-18 
---

我们先来看如下一段模板：

```html
<div id="app">
    <h2 v-text='hello' v-show='isShow'></h2>
    <input type="text" v-model='counter'>
    <button v-on-click='add' type="button">add</button>
    <p v-text='counter'></p>
    <p v-text='info.age'></p>
</div>
```

```js
var vm = new TinyVue({
    el: 'app',
    data: {
        counter: 1,
        hello: 'ahahah!',
        isShow: true,
        info: {
            age: 18
        }   
    },
    methods: {
        add: function () {
            vm.counter += 1;
            vm.info.age += 1;
        }
    },
    ready () {
        
    }
})
```

这里需要注意的是，`info.age`,在框架内部绑定的时候，绑定的是 key 是 `info.age`, 但取值的时候是不能按`this['info.age']`的形式来取的，即需要考虑对象的层次问题。


所以对于这种深层次的key,我们需要单独的处理，代码如下:

[代码出处](https://github.com/xiaofuzi/re-vue/blob/0c10af3987358985760cc8146b6e0e2f5adec02a/src/binding.js)
```js
defineRective (obj, path) {
    let self = this,
    key = path[0];

    if (path.length === 1) {
        def(obj, key, {
            get () {
                return self.value;
            },
            set (value) {
                if (value !== self.value) {
                    self.value = value;
                    self.update(value);
                }
            }
        });
    } else {
        let subObj = obj[key];
        if (!subObj) {
            subObj = {};

            def(obj, key, {
                get () {
                    return subObj;
                },
                set (value) {
                    objectEach(value, (key)=>{
                        subObj[key] = value[key];
                    });
                }
            });
        }
        self.defineRective(subObj, path.slice(1));
    }
}
```

```js
//path 
let key = 'info.page'
let path = key.split('.');      //['info', 'page']
```

这里根据path的长度来判断是单路径还是深层次路径，从而进行不同的处理。单层次路径的处理没什么特别的，重点看下深层次路径的处理。

```js
let subObj = obj[key];
if (!subObj) {
    subObj = {};

    def(obj, key, {
        get () {
            return subObj;
        },
        set (value) {
            objectEach(value, (key)=>{
                subObj[key] = value[key];
            });
        }
    });
}
```

注意看这里的 set 操作，这里对中间赋值时对其所有子节点进行赋值操作，如果其子节点的子节点还是对象那么会一直将该操作传递下去，直到跟节点，从而触发根节点绑定的指令更新操作(根节点才可以绑定指令)。

如下所示:
```js
//这里会正常的触发'info.page'所绑定的指令更新操作
this.info.page += 1;

/**
 * 这里我们直接给'info'赋值，如果没有我们之前的操作，那么'info.age'的更新  
 * 操作是不会触发的，因为我们还没有对'info'进行监测，但当我们进行如上的转换后，
 * 对'info'进行监测，当'info'的赋值操作发生时，触发'info'的子属性的赋值操作，
 * 从而触发子属性的更新,所以进行如下的操作也实现了'info.age'的指令更新
 */
this.info = {
    age: 24
}

/**
 * 附加说明，存储绑定过程的对象如下所示
 this._bindings = {
    'info.page': {
        value: 18,
        directives: []
    }
 }
 */
```

如下为对深层次 key 的取值和赋值的辅助函数，这里直接给 Object 扩展了 $get 和 $set 函数， 从而可以实现如下操作：

```js
let person = {
    info: {
        age: 18
    }
}

person['info.age'] = 24;
console.log(person.info.age);       //24
```

```js
/**
 * Object extend
 */
Object.prototype.$get = function (path='') {
    path = path.split('.');
    if (path.length == 1) {
        return this[path[0]];
    } else {
        this[path[0]] = this[path[0]] || {};
        return this[path[0]].$get(path.slice(1).join('.'));
    }
};

Object.prototype.$set = function (path='', value) {
    path = path.split('.');
    if (path.length == 1) {
        this[path[0]] = value;
    } else {
        this[path[0]] = this[path[0]] || {};
        this[path[0]].$set(path.slice(1).join('.'), value);
    }
};
```











