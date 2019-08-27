---
title: Proxy与Object.defineProperty
date: 2019-06-11 15:56:51
tags: 
    - Vue3.x
    - Proxy
    - Object.defineProperty
    - 数据劫持与双向绑定
categories:
    - 笔记
---
> 众所周知，`Vue2.x`采用``Object.defineProperty``的`getter`、`setter`进行数据劫持的;Vue3.x将采用`Proxy`替代`Object.defineProperty`,`Proxy`是`ES6`（`ES2015`）中新增的特性，也可以用做数据劫持。所谓数据劫持，指的是在访问或修改某个属性时，通过代码拦截这个行为，进行额外的操作或修改返回的结果。
> 数据劫持应用最多的就是双向绑定。

### Object.defineProperty
`Object.defineProperty`的主要问题有三个：
#### 1.无法监听数组变化
代码如下
```javascript
let arr = [1,2,3]
let obj = {}
Object.defineProperty(obj, 'arr', {
  get () {
    console.log('get arr')
    return arr
  },
  set (newVal) {
    console.log('set', newVal)
    arr = newVal
  }
})
obj.arr.push(4) // 只会打印 get arr, 不会打印 set
obj.arr = [1,2,3,4] // 这个能正常 set
```
数组的原生操作方法`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`不会触发`set`。`Vue`的做法是把这些方法重写来实现数组劫持，简单代码实现如下：
```javascript
const aryMethods = ['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'];
const arrayAugmentations = [];
aryMethods.forEach((method) => {

  // 原生Array的原型方法
  let original = Array.prototype[method];

  // 将 push, pop 等封装好的方法定义在对象 arrayAugmentations 的属性上
  // 注意：是实例属性而非原型属性
  arrayAugmentations[method] = function () {
    console.log('我被改变啦!');

    // 调用对应的原生方法并返回结果
    return original.apply(this, arguments);
  };
});
let list = ['a', 'b', 'c'];
// 将我们要监听的数组的原型指针指向上面定义的空数组对象
// 这样就能在调用 push, pop 这些方法时走进我们刚定义的方法，多了一句 console.log
list.__proto__ = arrayAugmentations;
list.push('d');  // 我被改变啦！
// 这个 list2 是个普通的数组，所以调用 push 不会走到我们的方法里面。
let list2 = ['a', 'b', 'c'];
list2.push('d');  // 不输出内容
```
<!-- more -->
#### 2.必须遍历对象的每个属性
使用 `Object.defineProperty`多数要配合`Object.keys()`和遍历, 于是多了一层嵌套。
```javascript
Object.keys(obj).forEach(key => {
  Object.defineProperty(obj, key, {
    // ...
  })
})
```
#### 3.必须深层遍历嵌套的对象
```javascript
let obj = {
  info: {
    name: 'spicy'
  }
}
```
这一类的嵌套对象,必须逐层遍历,直到把每个对象的每个属性都调用 `Object.defineProperty`为止,`Vue`源码中的方法叫`walk`。

### Proxy
#### 什么是`Proxy`
- `Proxy`是`ES6`新增的特性，意思是“代理”
- `Proxy`能够以简洁易懂的方式控制外部对对象的访问。其功能非常类似于设计模式中的代理模式。
- `Proxy`可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。
- 使用 `Proxy`的核心优点是可以交由它来处理一些非核心逻辑（如：读取或设置对象的某些属性前记录日志；设置对象的某些属性值前，需要验证；某些属性的访问控制等）。 从而可以让对象只需关注于核心逻辑，达到关注点分离，降低对象复杂度等目的。

#### 基本用法
```javascript
let p = new Proxy(target, handler);
```
- `target`是用`Proxy`包装的被代理对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）
- `handler`是一个对象，其声明了代理`target`的一些操作，其属性是当执行一个操作时定义代理的行为的函数。
- `p`是代理后的对象。当外界每次对`p`进行操作时，就会执行 `handler`对象上的一些方法。`Proxy`共有13种劫持操作，`handler`代理的一些常用的方法有如下几个

```text
get: 读取
set：修改
has：判断对象是否有该属性
construct：构造函数
```
**`Proxy`与`Object.defineProperty`对比:**
#### 1.针对对象
`Proxy`在数据劫持方面是`Object.defineProperty`的升级版，它是针对整个对象，而不是某个对象的属性，所以不需要对`keys`进行遍历。
```javascript
let obj = {
  name: 'spicy',
  age: 30
}
let handler = {
  get (target, key, receiver) {
    console.log('get', key)
    return Reflect.get(target, key, receiver)
  },
  set (target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  }
}
let Proxy = new Proxy(obj, handler)
Proxy.name = 'Zoe' // set name Zoe
Proxy.age = 18 // set age 18
```
[`Reflect`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)也是`ES6`新增的特性，与`Proxy`的`handler`对象拥有相同的方法，其中一些方法与`Object`的方法相同。

#### 2.支持数组
```javascript
let arr = [1,2,3]
let Proxy = new Proxy(arr, {
    get (target, key, receiver) {
        console.log('get', key)
        return Reflect.get(target, key, receiver)
    },
    set (target, key, value, receiver) {
        console.log('set', key, value)
        return Reflect.set(target, key, value, receiver)
    }
})
Proxy.push(4)
// get push     (寻找 `Proxy`.push 方法)
// get length   (获取当前的 length)
// set 3 4      (设置 `Proxy`[3] = 4)
// set length 4 (设置 `Proxy`.length = 4)
```
`Proxy`不需要对数组的方法进行重写，省去了众多`hack`，减少代码量等于减少了维护成本，而且标准的就是最好的。

#### 3.嵌套支持
本质上，`Proxy` 也是不支持嵌套的，这点和 `Object.defineProperty`是一样的。因此也需要通过逐层遍历来解决。`Proxy` 的写法是在`get`里面递归调用 `Proxy` 并返回，代码如下：
```javascript
let obj = {
  info: {
    name: 'spicy',
    blogs: ['webpack', 'babel', 'cache']
  }
}
let handler = {
  get (target, key, receiver) {
    console.log('get', key)
    // 递归创建并返回
    if (typeof target[key] === 'object' && target[key] !== null) {
      return new Proxy(target[key], handler)
    }
    return Reflect.get(target, key, receiver)
  },
  set (target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  }
}
let Proxy = new Proxy(obj, handler)
// 以下两句都能够进入 set
Proxy.info.name = 'Zoe'
Proxy.info.blogs.push('Proxy')
```
#### 其他
- [`Proxy`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/`Proxy`#handler_%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%96%B9%E6%B3%95)的第二个参数可以有13种拦截方法，比`Object.defineProperty`要更加丰富
- `Proxy` 的兼容性不如 `Object.defineProperty`, 不支持`QQ Browser`、`Baidu Browser`、`IE`等浏览器。[`Object.defineProperty`](https://caniuse.com/#search=defineProperty)除了低版本的IE不支持外，其他几乎都是支持的。
- 不能使用`polyfill`来处理兼容性

### 基于`Proxy`实现的双向绑定
`html`结构：
```javascript
<div id="app">
      <input type="text" id="input" />
      <div>您输入的是： <span id="title"></span></div>
      <button type="button" name="button" id="btn">添加到toDoList</button>
      <ul id="list"></ul>
 </div>
```
定义一个`Proxy`, 实现输入框的双向绑定显示:
```javascript
const obj = {};
const input = document.getElementById("input");
const title = document.getElementById("title");

const newObj = new Proxy(obj, {
    get: function(target, key, receiver) {
        console.log(`getting ${key}!`);
        return Reflect.get(target, key, receiver);
    },
    set: function(target, key, value, receiver) {
        console.log(target, key, value, receiver);
        if (key === "text") {
            input.value = value;
            title.innerHTML = value;
        }
        return Reflect.set(target, key, value, receiver);
    }
});

input.addEventListener("keyup", function(e) {
    newObj.text = e.target.value;
});
```
添加`toDoList`列表，把数组渲染到页面上去:
```javascript
// 渲染toDoList列表
const Render = {
    init: function(arr) {
    const fragment = document.createDocumentFragment();
        for (let i = 0; i < arr.length; i++) {
            const li = document.createElement("li");
            li.textContent = arr[i];
            fragment.appendChild(li);
        }
        list.appendChild(fragment);
    },
    addList: function(val) {
        const li = document.createElement("li");
        li.textContent = val;
        list.appendChild(li);
    }
};

```
再定义一个`Proxy`，实现`toDoList`的添加:
```javascript
const arr = [];
// 监听数组
const newArr = new Proxy(arr, {
    get: function(target, key, receiver) {
        return Reflect.get(target, key, receiver);
    },
    set: function(target, key, value, receiver) {
        console.log(target, key, value, receiver);
        if (key !== "length") {
            Render.addList(value);
        }
        return Reflect.set(target, key, value, receiver);
    }
});

// 初始化
window.onload = function() {
    Render.init(arr);
};

btn.addEventListener("click", function() {
    newArr.push(newObj.text);
    newObj.text = '' //置空文本框
});
```
运行效果如下：
<div id="app">
    <input type="text" id="input" />
    <div>您输入的是： <span id="title"></span></div>
    <button type="button" name="button" id="btn">添加到toDoList</button>
    <ul id="list"></ul>
</div>
<script>
    const obj = {};
    const input = document.getElementById("input");
    const title = document.getElementById("title");
    const newObj = new Proxy(obj, {
        get: function(target, key, receiver) {
            console.log(`getting ${key}!`);
            return Reflect.get(target, key, receiver);
        },
        set: function(target, key, value, receiver) {
            console.log(target, key, value, receiver);
            if (key === "text") {
                input.value = value;
                title.innerHTML = value;
            }
            return Reflect.set(target, key, value, receiver);
        }
    });
    input.addEventListener("keyup", function(e) {
        newObj.text = e.target.value;
    });
    // 渲染toDoList列表
    const Render = {
        init: function(arr) {
        const fragment = document.createDocumentFragment();
            for (let i = 0; i < arr.length; i++) {
                const li = document.createElement("li");
                li.textContent = arr[i];
                fragment.appendChild(li);
            }
            list.appendChild(fragment);
        },
        addList: function(val) {
            const li = document.createElement("li");
            li.textContent = val;
            list.appendChild(li);
        }
    };
    const arr = [];
    // 监听数组
    const newArr = new Proxy(arr, {
        get: function(target, key, receiver) {
            return Reflect.get(target, key, receiver);
        },
        set: function(target, key, value, receiver) {
            console.log(target, key, value, receiver);
            if (key !== "length") {
                Render.addList(value);
            }
            return Reflect.set(target, key, value, receiver);
        }
    });
    // 初始化
    window.onload = function() {
        Render.init(arr);
    };
    btn.addEventListener("click", function() {
        newArr.push(newObj.text);
        newObj.text = ''
    });
</script>

> 参考文档
> [数据劫持 OR 数据代理](https://mp.weixin.qq.com/s/SPoxin9LYJ4Bp0goliEaUw)