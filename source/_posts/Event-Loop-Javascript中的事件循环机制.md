---
title: Event Loop--Javascript中的事件循环机制
date: 2019-06-06 11:27:02
tags:
    - Javascript
    - Event Loop
categories:
    - 笔记
---
## 前言
`Event Loop`即事件循环，是在浏览器或`Node`的一种解决`JavaScript`单线程运行时不会阻塞的运行机制，也是在`JavaScript`中使用异步的原理。

### 为什么`Javascript`是单线程的？
众所周知，`Javascript`是单线程的，也就是说，同一时间只能做一件事。`Javascript`作为浏览器脚本语言，主要用来处理与用户的的交互、网络请求以及操作`DOM`。
假设在同一时间，两个线程分别对同一`DOM`进行删除、修改，这时浏览器无法判断两个线程执行的先后顺序，为了避免出现这种问题，`Javascript`必须是单线程的。
`Javascript`是单线程的，在主线程中依次执行所有操作任务，如果某个任务是个耗时任务（比如请求某个接口、加载高清图片），这个任务不执行完后面的任务就不能执行，使线程阻塞。为了防止线程阻塞，于是有了异步的概念。

#### `Javascript`异步执行的运行机制
1.所有任务都在主线程上执行，形成一个执行栈。
2.主线程之外，还存在一个"任务队列"（`task queue`）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
3.一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"。那些对应的异步任务，结束等待状态，进入执行栈并开始执行。
4.主线程不断重复上面的第三步
<!-- more -->

#### 宏任务与微任务
异步任务分为宏任务(`macrotask`)与微任务(`microtask`)，不同的`API`注册的任务会依次进入自身对应的队列中，然后等待 `Event Loop `将它们依次压入执行栈中执行。
**宏任务(`macrotask`)**
`script`(整体代码)、`setTimeout`、`setInterval`、`UI` 渲染、 `I/O`、`postMessage`、` MessageChannel`、`setImmediate`(`Node.js` 环境)
**微任务(`microtask`)**
`Promise`、 `MutaionObserver`、`process.nextTick`(`Node.js`环境）

### Event Loop(事件循环)
`Event Loop`(事件循环)中，每一次循环称为 `tick`, 每一次`tick`的任务如下：
- 执行栈选择最先进入队列的宏任务(通常是`script`整体代码)，如果有则执行
- 检查是否存在 `Microtask`，如果存在则不停的执行，直至清空 `microtask` 队列
- 更新`render`(每一次事件循环，浏览器都可能会去更新渲染)
- 重复以上步骤
  
宏任务 > 所有微任务 > 宏任务，如图所示：
{% asset_img loop.png%}

**举个栗子**
```javascript
console.log('script start'); // 1

setTimeout(function() {
  console.log('timeout1'); // 5
}, 10);

new Promise(resolve => {
    console.log('promise1'); // 2
    resolve();
    setTimeout(() => console.log('timeout2'), 10); // 6
}).then(function() {
    console.log('then1') // 4
})

console.log('script end'); // 3
```
- 首先事件循环从宏任务(`macrotask`)队列开始，也就是`script`(整体代码), 打印出`script start`
- 接下来将`timeout1`放入`macrotask`队列[`timeout1`]
- 遇到`promise`，打印出`promise1`,将`timeout2`放入`macrotask`队列，[`timeout1`, `timeout2`]
- 将`then`放入`microtask`队列，[`then1`]
- 打印`script end`, 第一个宏任务执行完毕
- 从`microtask`队列取出`then1`，打印`then1`, 微任务执行完毕
- 从`macrotask`队列依次取出`timeout1`, `timeout2`, 打印`timeout1`, `timeout2`

### 当 `Event Loop` 遇到 `async/await`
`async/await`只是语法糖，转成promise的形式即可：
```javascript
async function foo() {
  // await 前面的代码
  await bar();
  // await 后面的代码
}

async function bar() {
  // do something...
}

foo();
```
其中`await`前面的代码是同步的，调用此函数时会直接执行；而 `await bar()`可以被转换成`Promise.resolve(bar())`；`await`后面的代码则会被放到`Promise`的`then()`方法里。转化结果如下：
```javascript
function foo() {
  // await 前面的代码
  Promise.resolve(bar()).then(() => {
    // await 后面的代码
  });
}

function bar() {
  // do something...
}

foo();
```
**举个栗子**
```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(function() {
  console.log('setTimeout');
}, 0);
async1();
new Promise(function(resolve) {
  console.log('promise1');
  resolve();
}).then(function() {
  console.log('promise2');
});
console.log('script end');
```
`async/await`转化成`promise`, 如下：

```javascript
function async1() {
  console.log('async1 start'); // 2

  Promise.resolve(async2()).then(() => {
    console.log('async1 end'); // 6
  });
}

function async2() {
  console.log('async2'); // 3
}

console.log('script start'); // 1

setTimeout(function() {
  console.log('timeout1'); // 8
}, 0);

async1();

new Promise(function(resolve) {
  console.log('promise1'); // 4
  resolve();
}).then(function() {
  console.log('promise2'); // 7
});
console.log('script end'); // 5
```
- 首先打印出`script start`
- 接着将`timeout1`加入`macrotask`队列，[`timeout1`]
- 执行`async1`，打印`async1 start`，遇到`Promise.resolve`, 执行`async2`，打印`async2`，将`async1 end`加入`microtask`队列，[`async1 end`]
- 遇到`promise`, 打印`promise1`，将`promise2`加入`microtask`队列， [`async1 end`, `promise2`]
- 打印`script end`
- 执行`microtask`队列, 依次打印出`async1 end`, `promise2`
- 执行`macrotask`，打印`timeout1`