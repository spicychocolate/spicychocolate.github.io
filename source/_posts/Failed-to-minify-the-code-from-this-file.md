---
title: Failed to minify the code from this file
date: 2019-05-30 10:40:46
tags:
    - React
    - create-react-app
categories:
    - 踩坑
---
**用`create-react-app`生成的项目运行`npm run build`命令报以下错误**
{% asset_img error.png %}
`Failed to minify the code from this file`, 不能压缩`source-map`的代码
<!-- more -->
解决方式：
**1.将`source-map`以第三方插件的方式在`index.html`引入**
```javascript
// index.html
<script src="https://unpkg.com/source-map@0.7.3/dist/source-map.js"></script>
```
**2.修改`webpack`配置,添加外部依赖配置**
```javascript
// webpack.config.js
externals: {
    'source-map': 'sourceMap'
}
```
**3.在代码中引用**
```typescript
import * as SourceMap from 'source-map'
```
以上