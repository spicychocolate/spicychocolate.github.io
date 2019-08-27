---
title: 使用Yeoman搭建前端脚手架
date: 2019-08-06 14:33:43
tags:
    - Yeoman
---
## 为什么使用脚手架？
大家都晓得，使用脚手架可帮助我们快速构建项目。我们可以通过[Yeoman](http://yeoman.io/)搭建一个自己的脚手架，然后发布到npm或公司内部搭建的仓库，供小伙伴使用。
附一个已搭建好的脚手架：https://github.com/spicychocolate/generator-spicychocolate

## 如何搭建一个脚手架

### 安装工具
```bash
npm install -g yo
npm install -g generator-generator // 脚手架生成工具
```

### 初始化脚手架
```bash
yo generator
```
> 注意：初始化时要指定keyswords为yeoman-generator, 或手动在package.json添加

<!-- more -->

生成的目录结构如下:
```javascript
generator-spicychocolate
├── LICENSE
├── README.md
├── __tests__
│   └── app.js
├── generators
│   └── app
│       ├── index.js
│       └── templates
│           └── dummyfile.txt
└── package.json
```

- generators/app/templates/是默认存放文件的目录，把所有模版文件放在这个目录下
- generators/app/index.js是Yeoman的配置文件，定义如何生成脚手架

### 编写脚手架
脚手架所做的事
- 接收用户输入
- 根据用户输入生成模板文件
- 将模板文件拷贝到目标目录（通常是用户运行脚手架的目录）
- 安装依赖

`yeoman` 提供了一个基本生成器，你可以扩展它以实现自己的行为。这个基础生成器将帮你减轻大部分工作量。在生成器的 index.js 文件中，以下是扩展基本生成器的方法：

```javascript
var Generator = require("yeoman-generator");
module.exports = class extends Generator {};
```
`yeoman` 生命周期函数执行顺序如下：

```javascript
initializing - 初始化函数
prompting - 接收用户输入阶段
configuring - 保存配置信息和文件
default - 执行自定义函数
writing - 生成项目目录结构阶段
conflicts - 统一处理冲突，如要生成的文件已经存在是否覆盖等处理
install - 安装依赖阶段
end - 生成器结束阶段
```
常用的是`initializing`、`prompting`、`default`、`writing`、`install` 这四种生命周期函数。

代码如下：
```javascript
"use strict";
const Generator = require("yeoman-generator");
const chalk = require("chalk"); // 让console.log带颜色输出
const yosay = require("yosay");
const mkdirp = require("mkdirp"); // 创建目录

module.exports = class extends Generator {
  initializing() {
    this.props = {};
  }
  
  // 接受用户输入
  prompting() {
    this.log(
      yosay(
        `Welcome to the grand ${chalk.red(
          "generator-spicychocolate"
        )} generator!`
      )
    );

    const prompts = [
      {
        type: "input",
        name: "projectName",
        message: "请输入项目名称："
      }
    ];

    return this.prompt(prompts).then(props => {
      this.props = props;
    });
  }

  // 创建项目目录
  default() {
    const {projectName} = this.props;
    if (path.basename(this.destinationPath()) !== projectName) {
      this.log(`\nYour generator must be inside a folder named
        ${this.props.name}\n
        I will automatically create this folder.\n`);

      mkdirp(projectName);
      this.destinationRoot(this.destinationPath(projectName));
    }
  }
  // 写文件
  writing() {
    // 将templates目录的代码拷贝到目标目录
    // templates目录默认路径是generators/app/templates
    this.fs.copy(
      this.templatePath("dummyfile.txt"),
      this.destinationPath("dummyfile.txt")
    );
    this._writingPackageJSON();
  }

  // 以下划线_开头的是私有方法
  _writingPackageJSON() {
    // this.fs.copyTpl(from, to, context)
    this.fs.copyTpl(
      this.templatePath("_package.json"),
      this.destinationPath("package.json"),
      {
        projectName: this.props.projectName// context上下文可替换模板中的ejs语法变量<%= projectName %>
      }
    );
  }

  // 安装依赖
  install() {
    this.installDependencies();
  }
};
```

### 运行测试用例
用generator生成的脚手架已经有测试用例，在其基础上修改之后代码如下：

```javascript
'use strict';
const path = require('path');
const assert = require('yeoman-assert');
const helpers = require('yeoman-test');

describe('generator-spicychocolate:app', () => {
  beforeAll(() => {
    return helpers
      .run(path.join(__dirname, '../generators/app'))
      .withPrompts({ someAnswer: true });
  });

  it('creates files', () => {
    assert.file(['dummyfile.txt']);
    assert.file(['package.json']);
  });
});
```
### 运行脚手架
编写完脚手架之后，在本地运行调试，可通过npm创建全局模块并将其符号链接到本地模块。在根目录运行：
```bash
npm link
```
然后就可以使用`yo spicychocolate`运行脚手架

## 发布
编写好的脚手架可发布到npm或公司内部搭建的包管理仓库，npm发布如下：
```
npm addUser // 需将npm源切换到https://registry.npmjs.org/
npm login
npm publish // 每次发布需更改package.json的版本号
```

> 参考链接： 
> [用 yeoman 打造自己的项目脚手架](https://juejin.im/post/5b7a24a5e51d4538da22d055)
yyaf
