---
title: 认识dart
date: 2019-11-29 10:05:26
tags:
  - dart
---

> `Dart`是谷歌开发的计算机编程语言。用于`web`、服务器、移动应用和物联网等领域的开发。也是开源的软件。
> `Dart`设计的时候就吸取了其他语言（`Java`、`C`、`JavaScript`等） 的各种优点， `Dart`非常容易使用。
> `Dart`是面向对象的、类定义的、单继承的语言。它的语法类似 c 语言，支持接口`interface`、混入`mixins`、抽象类`abstract classes`、具体化泛型`reified generics`、可选类型`option typing` 和 `sound type system`。
> `Dart2`成为强类型语言
> `Dart2`有三大运行环境, 分别是`Flutter`, `Web`和`Server`. 其中`Flutter`框架自带了`Dart`语言, 可以用于开发跨平台的`iOS`和`Android`应用. 而`Web`则是通过内置的`dart2js`命令将`Dart`转译为`JavaScript`语言后在浏览器环境运行.`Server`的目标是使用`Dart`语言开发后端服务程序, 程序通过`Dart`虚拟机运行.

### 安装 dart

1. 官网下载[`dart-sdk`](https://dart.dev/tools/sdk/archive)
2. 解压文件，将`dart-sdk`文件夹下的`bin`目录添加到环境变量
3. 打开命令行运行`dart`命令检查是否安装成功
   {% asset_img dart.png%}

### 第一行 dart

新建`demo.dart`文件，打开`IED`,输入以下代码：

```dart
// demo.dart
void main(){
    print('hello word');
}
```

然后再命令行中执行`dart demo.dart`:

{% asset_img hello_word.png%}

`Dart`和绝大多数编译型语言一样，都要求`main`作为语言执行的入口(`entrypoint`)，而不是像`JavaScript`那样解释执行。所以显而易见，`Dart`是编译型语言。
但我们在运行`demo.dart`时并没有先编译，这是因为`Dart`通过`VM`来运行，在载入源码时先编译成字节码，再通过字节码执行，这样的好处是执行速度会比`JIT`(`Just in Time`)代码快很多，弊端就是代码的载入到真正执行的速度就比较慢了。所以，Dart 也支持编译成`AOT`(`Ahead of Time`)代码，这样代码载入运行的速度能得到很大的提升。

### `Dart`语法初探

#### 变量和类型

在 `Dart`中，我们可以用`var`或者具体的类型来声明一个变量：

```dart
var name = 'Bob';
int num = 9527;
```

默认情况下，未初始化的变量的值都是 `null`

```dart
int count;
print(count == null); 	// true
```

这样我们就不用担心在一些情况下无法断定一个传递过来的变量到底是 `undefined` 还是`null`了。因为在`Dart`中，所有未定义的情况都是`null`。

`Dart`是类型安全的语言，并且所有类型都是对象类型，所有的类型的顶层都来自于 `Object`，理所当然地，`null`也是一个对象。所以，`var`关键字只是一个类型的引用，使用它所声明的对象只是该对象类型的一个引用，这个过程由编译器完成，并不需要关心。
`Dart`拥有比较简单的内置类型，如 `Number`, `String`, `Boolean`, `List`, `Map`, `Runes`, `Symbols`。

#### 函数

函数是`Dart`执行的基本单元，在函数外的表达式不能获得正确的执行。
一个基本的函数如下：

```dart
void sayHello() {
  print('hello');
}
```

如果返回值是`void`，那么`void`是可以省略的。
如果你的函数体只有一行返回值，你可以用**箭头函数**：

```dart
bool isZero(int num) => num == 0;
```

函数其实也是一个对象，你可以把函数作为参数传递或者赋值：

```dart
var loudify = (msg) => '!!! ${msg.toUpperCase()} !!!';
print(loudify('hello') == '!!! HELLO !!!');
```

也可以用**匿名函数**来迭代 List:

```dart
var list = ['apples', 'bananas', 'oranges'];
list.forEach((item) {
  print('${list.indexOf(item)}: $item');
});
```

#### 运算符

**`?.`运算符：**假设`Person`这个类有`sayHello()`函数，`p`是`Person`的一个实例。那么 `p?.sayHello()` 表示 `p` 为 `null`的时候避免抛出 `exception`。
**`is`运算符：**如果 `p`是 `Person`这个类的实例，那么 `p is Person`返回 `true`。
**`??`运算符：**`a ?? b`表达式表示如果 `a`不为 `null`，返回 `a`的值，否则返回 `b`。
**`??=`运算符：**`b ??= value`表示如果 `b`为 `null`则给 `b`赋值 `value`，否则不赋值。

#### 流程控制

`Dart`保持了标准的流程控制语法：`if-else`, `for`, `while`, `do-while`, `break/continue`, `switch-case`, `assert`.
值得一提的是 `assert`，它可以阻止你的代码执行流程（和`C`中的 `assert`类似），不过它只在 `Dart`的 `check mode`有用（类似于开发环境），在 `production mode`无效。

#### 类

在`Dart`中，声明并使用一个类很简单：

```dart
class Point {
  num x;
  num y;
}

void main() {
  var point = new Point();
  point.x = 4;
  assert(point.x == 4);
  assert(point.y == null);
}
```

**构造函数**：

```dart
class Point {
  num x, y;

  // 常见的标准构造函数
  Point(num x, num y) {
    this.x = x;
    this.y = y;
  }

  // 语法糖，作用和上面一样
  Point(this.x, this.y);

  // 可以命名一个特殊的构造函数
  Point.origin() {
  	x = 0;
  	y = 0;
  }

  // 初始化成员列表，这里不能使用 this关键字给 x/y 赋值
  Point()
    : x = 0,
      y = 0 {
    print('初始化成员列表');
  }
}
```

值得一提的是，`Dart` 没有 `public`、`private`这些关键字，方法名前面加上 `_`即可作为 `private`方法使用。默认都是 `public`。
`Dart`还支持抽象类

```dart
abstract class AbstractContainer {
  // 这里可以定义构造函数、成员变量、方法等待
  void updateChildren(); // 抽象方法.
}

class SpecializedContainer extends AbstractContainer {
  void updateChildren() {
    // 这里提供方法的实现，这样这个方法就不是抽象的了
  }
  // 未实现的抽象方法会抛一个 warning，但并不妨碍对象实例化
}
```

#### 错误处理

```dart
// demo
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // 特定类型的 exception
  buyMoreLlamas();
} on Exception catch (e) {
  // 不确定什么类型的 exception
  print('Unknown exception: $e');
} catch (e) {
  // 没有类型，统一处理
  print('Something really unknown: $e');
}
```

#### 库

一门编程语言的生态取决于它的库，来看一看 `Dart`库导入的语法。

```dart
// 导入 html库
import 'dart:html';

// 只导入一部分
import 'package:test/test.dart';

// 给个 alias
import 'package:lib2/lib2.dart' as lib2;

// 只导入该库中的 foo
import 'package:lib1/lib1.dart' show foo;

// 除了 foo全都导入
import 'package:lib2/lib2.dart' hide foo;

// 懒加载
import 'package:greetings/hello.dart' deferred as hello;
```

### Dart 的特性

- 单进程异步事件模型；
- 强类型，可以类型推断；
- `DartVM`，具有极高的运行效率和优秀的代码运行优化，根据早前的基准测试，性能比肩 `Java7` 的`JVM`；
- 独特的隔离区( `Isolate` )，可以实现多线程；
- 面向对象编程，一切数据类型均派生自 `Object` ；
- 运算符重载，泛型支持；
- 强大的 `Future` 和 `Stream` 模型,可以简单实现高效的代码；
- `Mixin`特性，可以更好的实现方法复用；
- 全平台语言，可以很好的胜任移动和前后端的开发。
- 在语法上，`Dart` 提供了很多便捷的操作，可以明显减少代码量。比如字符连接，可以直接 `"my name is $name, age is $age"`，无需+号拼接，也无需做类型转换。

### Dart 与 Javascript 的简单比较

1.`Dart`只有一种否定条件
在 JS 中，你可以用 `false`, `null`, `undefined`,`""`,`0`,`NaN` 作为否定条件:

```javascript
var a = null;
if (!a) {
  // do
}
```

在`Dart`中，只有 `false` 才是否定条件，上面的代码必须写成这样:

```dart
var a =null;
if(a != null) {
    // do
}
```

2.`Dart`是强类型语言，可以开启类型检查

3.解析`DOM`
`js`获取`dom`的方法如下：

```javascript
getElementsById();
getElementsByTagName();
getElementsByName();
getElementsByClassName();
querySelector();
querySelectorAll();
document.links;
document.images;
document.forms;
document.scripts;
formElement.elements;
selectElement.options;
```

`Dart`学习了`JQuery`，只有两个方法:

```dart
elem.query('#foo');
elem.queryAll('.foo');
```

4.`undefined` 和 `null`
在`JS`中，这两个值大多数情况可以互换，但有时候又不是，让人很头疼。`Dart`只有`null`

5.获取属性值安全处理
js 需要判断对象存在才能进一步获取属性值，属性层级越深，判断比较越多

```js
var name =
  person.address || person.address.street || person.address.street.name;
```

在 dart 可以通过`?.`操作符简单获取：

```dart
// dart
var name = address?.street?.name;
```

6.级联运算符
在 `Javascript` 中，数组 `push` 函数返回数组的长度

```js
// js
var parks = [1, 2, 3];
parks.push(4, 5);
console.log(parks); // [1, 2, 3, 4, 5]

var shelters = [1, 2, 3];
shelters[1] = 4;
shelters[2] = 5;
console.log(shelters); // [1, 4, 5]
```

在 `Dart` 中，我们有级联运算符列表`..Add()` ，它允许我们返回列表:

```dart
// dart
main() {
  print([1, 2, 3]..add(4)..add(5));  // [1, 2, 3, 4, 5]
  print([1, 2, 3]..[1]=4..[2]=5);  // [1, 4, 5]
}
```

> 参考链接：
> https://www.stephenw.cc/2018/04/20/dart-getting-start/
