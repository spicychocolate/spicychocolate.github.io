---
title: js常用设计模式（一）
date: 2019-10-17 16:52:32
tags: 
    - JS
    - 设计模式
---
## js常用设计模式（一）
- 单例模式
- 工厂模式
- 策略模式

### 单例模式
**概念：**
    单例模式是指一个类只有一个实例，多次实例化同一个类，得到的实例都是同一个。

**实现：**
```javascript
class Singleton{
    constructor(name){
        this.name = name;
    }
    getName(){
        return this.name;
    }
}
var getInstance = (function(){
    var instance = null;
    return function(name){
        if(!instance){
            instance = new Singleton(name);
        }
        return instance;
    }
})();

var a = getInstance('aa');
var b = getInstance('bb');
console.log(b.getName()); // "aa"
console.log(a === b); // true
```
**运用场景：实现一个弹窗**
```javascript
const createDialog = (function(){
    const dialog = null;
    return function(){
        return dialog || (dialog = document.body.appendChild(document.createElement('div')));
    }
})();
```
上面的代码还有缺点，它只能用于创建弹窗，假如又需要写一个单例函数，比如一个唯一的`xhr`对象，因此我们可以实现一个通用的`singleton`包装器，最终代码如下：
```javascript
const singleton = (function(fn){
    const result;
    return function(){
        return result || (result = fn.apply(this, arguments));
    }
})();

const createDialog = singleton(function(){
    return document.body.appendChild(document.createElement('div'));
});

```

### 工厂模式
**概念：**
工厂方法模式是一种实现了“工厂”概念的面向对象设计模式。工厂方法模式的实质是“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。

**例子:**
我们可以使用Object构造函数来创建单个对象，但是，使用同一个接口创建很多对象时，会产生大量重复的代码。为了解决这个问题，我们可以使用工厂模式。
```javascript
var employee1 = new Object();
employee1.position = "Front end engineer";
employee1.tool = "I love vscode.";
employee1.introduction = function () {
    console.log("I am a " + this.position + ", and " + this.tool);
}
var employee2 = new Object();
employee2.position = "UI designer";
employee2.tool = "I love photoshop.";
employee2.introduction = function () {
    console.log("I am a " + this.position + ", and " + this.tool);
}
employee1.introduction();//I am a Front end engineer, and I love vscode.
employee2.introduction();//I am a UI designer, and I love photoshop.
```
在上边这个例子中，我们定义了两个employee，一个是Front End Engineer，另一个是UI designer，他们都有position属性和tool属性，也都有introduction方法。如果我们需要创建很多个类似employee的对象呢，那我们就需要重复很多类似的代码。接下来，我们做一些简单的修改：
```javascript
function Employee(type) {
    var employee;
    if (type == "programmer") {
        employee = new Programmer();
    } else if (type == "designer") {
        employee = new Designer();
    }
    employee.introduction = function () {
        console.log("I am a " + this.position + ", and " + this.tool);
    }
    return employee;
}

function Programmer() {
    this.position = "Front end engineer";
    this.tool = "I love vscode.";
}
function Designer() {
    this.position = "UI designer";
    this.tool = "I love photoshop.";
}

var employee1 = Employee("programmer");
employee1.introduction();//I am a Front end engineer, and I love vscode.
var employee2 = Employee("designer");
employee2.introduction();//I am a UI designer, and I love photoshop
```
在上边这段代码中，我们将employee的初始化分别放到了Programmer()和Designer()中实现。这其实就是一个简单工厂模式的例子，Employee是一个工厂，可以根据传入的type的不同，创建不同的employee，每个employee有自己的职位和使用的工具，每个employee都可以介绍自己的这些信息。
**实际例子**
- `jQuery`的`$(selector)`
```javascript
class jQuery {
    constructor(selector) {
        super(selector)
    }
    //  ....
}

window.$ = function(selector) {
    return new jQuery(selector)
}
```
- `React`的`createElement()`
```javascript
React.createElement('img', {src: 'avatar.png', className: 'avatar'});
React.createElement('h1', null, 'title');
```
### 策略模式
**定义：**
策略模式定义一系列的算法，分别封装起来，让他们之间可以互相替换
**例子：**
在一个Web项目中，注册、登录等功能的实现都离不开表单提交。表单校验也是前端常常需要做的事。假设我们正在编写一个注册的页面，在点击提交按钮之前，有如下几条校验逻辑:
- 用户名不可为空，不允许以空白字符命名，用户名长度不能小于2位。
- 密码长度不能小于6位。
- 正确的手机号码格式。

一般的实现方式，如下：
```html
<html>
<head>
    <title>策略模式-校验表单</title>
    <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
</head>
<body>
    <form id = "registerForm" method="post" action="http://xxxx.com/api/register">
        用户名：<input type="text" name="userName">
        密码：<input type="text" name="password">
        手机号码：<input type="text" name="phoneNumber">
        <button type="submit">提交</button>
    </form>
    <script type="text/javascript">
       var registerForm = document.getElementById('registerForm');
       registerForm.onsubmit = function() {
           if (registerForm.userName.value === '') {
               alert('用户名不可为空');
               return false;
           }
           if (registerForm.userName.value === '') {
               alert('用户名不可为空');
               return false;
           } 
           if (registerForm.userName.value.trim() === '') {
               alert('用户名不允许以空白字符命名');
               return false;
           } 
           if (registerForm.userName.value.trim().length < 2) {
               alert('用户名用户名长度不能小于2位');
               return false;
           } 
           if (registerForm.password.value.trim().length < 6) {
               alert('密码长度不能小于6位');
               return false;
           }
           if (!/^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|17[7]|18[0|1|2|3|5|6|7|8|9])\d{8}$/.test(registerForm.phoneNumber.value)) {
               alert('请输入正确的手机号码格式');
               return errorMsg;
             } 
       }
    </script>
</body>
</html>
```
这是一种很常见的编码方式，但它有很明显的缺点：
- registerForm.onsubmit 函数比较庞大，包含了很多if语句，这些语句要覆盖所有的校验规则。
- 若校验规则有变，不得不深入到registerForm.onsubmit 函数的内部实现，违反开放-封闭原则。
- 算法的复用性差。

下面，让我们来用策略模式重构表单校验:
```html
<html>
<head>
    <title>策略模式-校验表单</title>
    <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
</head>
<body>
    <form id = "registerForm" method="post" action="http://xxxx.com/api/register">
        用户名：<input type="text" name="userName">
        密码：<input type="text" name="password">
        手机号码：<input type="text" name="phoneNumber">
        <button type="submit">提交</button>
    </form>
    <script type="text/javascript">
        // 策略对象
        var strategies = {
            isNoEmpty: function (value, errorMsg) {
                if (value === '') {
                    return errorMsg;
                }
            },
            isNoSpace: function (value, errorMsg) {
                if (value.trim() === '') {
                    return errorMsg;
                }
            },
            minLength: function (value, length, errorMsg) {
                if (value.trim().length < length) {
                    return errorMsg;
                }
            },
            maxLength: function (value, length, errorMsg) {
                if (value.length > length) {
                    return errorMsg;
                }
            },
            isMobile: function (value, errorMsg) {
                if (!/^(13[0-9]|14[5|7]|15[0|1|2|3|5|6|7|8|9]|17[7]|18[0|1|2|3|5|6|7|8|9])\d{8}$/.test(value)) {
                    return errorMsg;
                }                
            }
        }

        // 验证类
        var Validator = function() {
            this.cache = [];
        }
        Validator.prototype.add = function(dom, rules) {
            var self = this;
            for(var i = 0, rule; rule = rules[i++];) {
                (function(rule) {
                   var strategyAry = rule.strategy.split(':');
                   var errorMsg = rule.errorMsg;
                   self.cache.push(function() {
                        var strategy = strategyAry.shift();
                        strategyAry.unshift(dom.value);
                        strategyAry.push(errorMsg);
                        return strategies[strategy].apply(dom, strategyAry);
                   })
                })(rule)
            }
        };
        Validator.prototype.start = function() {
            for(var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
                var errorMsg = validatorFunc();
                if (errorMsg) {
                    return errorMsg;
                }
            }
        };

        // 调用代码
        var registerForm = document.getElementById('registerForm');

        var validataFunc = function() {
            var validator = new Validator();
            validator.add(registerForm.userName, [{
                strategy: 'isNoEmpty',
                errorMsg: '用户名不可为空'
            }, {
                strategy: 'isNoSpace',
                errorMsg: '不允许以空白字符命名'
            }, {
                strategy: 'minLength:2',
                errorMsg: '用户名长度不能小于2位'
            }]);
            validator.add(registerForm.password, [ {
                strategy: 'minLength:6',
                errorMsg: '密码长度不能小于6位'
            }]);
            validator.add(registerForm.phoneNumber, [{
                strategy: 'isMobile',
                errorMsg: '请输入正确的手机号码格式'
            }]);
            var errorMsg = validator.start();
            return errorMsg;
        }

        registerForm.onsubmit = function() {
            var errorMsg = validataFunc();
            if (errorMsg) {
                alert(errorMsg);
                return false;
            }
        }
    </script>
</body>
</html>
```
策略模式是一种常用且有效的设计模式。
- 策略模式可以有效避免多重条件选择语句。
- 策略模式提供了对开放-封装原则的完美支持，将方法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展。
- 复用性高。

> 参考链接：
> https://segmentfault.com/a/1190000006899198