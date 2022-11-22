---
title: core-js 及 polyfill 理念
date: 2022-11-16T00:00:00.000+00:00
lang: zh
duration: 15min
---

什么是 `core-js`?

**core-js** 是一个 **JavaScript** 标准库，它包含了 **ECMAScript 2020** 在内的多项特性的 **polyfills**，以及 **ECMAScript** 在 **proposals** 阶段的特性、**WHATWG/W3C** 新特性等。因此它是一个现代化前端项目的“标准套件”。

总述：

`core-js` 是可以窥见前端工程化方方面面的一扇大门

`core-js` 又和 `Babel` 深度绑定，学习 `core-js` 可以更好地理解 `Babel` 生态，加深对前端生态的理解。

通过对 `core-js` 的解析，梳理前端一个极具特色的概念 — `polyfill(垫片/补丁)`

5个核心子包：

- `core-js`
- `core-js-pure`
- `core-js-compat`
- `core-js-bundle`
- `core-js-builder`

基础垫片能力是 `core-js` 的核心。

`core-js-pure` 提供了不污染全局变量的垫片能力

```jsx
import _from from 'core-js-pure/features/from';
import _flat from 'core-js-pure/features/array/flat';
```

`core-js-compat` 维护了按照 `browserslist`规范的垫片需求数据

```jsx
const {
  list, // array of required modules
  targets // object with targets for each module
} = require('core-js-compat')({
  targets: '>2.5%'
})
```

`core-js-compat` 被 `babel` 生态使用，由 Babel 分析出环境需要按需加载的垫片

`core-js-builder` 结合 `core-js-compat` 和 `core-js` 并利用 webpack 能力，根据需求打包出 `core-js` 代码

```jsx
require('core-js-builder')({
  targets: '>2.5%',
  filename: './my-core-js-bundle.js'
}).then(code => {}).catch(error => {})
```

`core-js-builder` 被 Node.js 服务使用，构建出不同场景的垫片包

复用一个 polyfill:

例如，复用数组方法 `Array.prototype.every`：

`函数签名` 如下：

```jsx
arr.every(callback(element[, index][, array])[, thisArg])
```

**考察：**`Array.prototype.every` 的底层实现。

发现宝藏：[https://tc39.es/ecma262/#sec-array.prototype.every](https://tc39.es/ecma262/#sec-array.prototype.every)

`tc39` 针对每一个方法的实现逻辑都有描述

babel更新博客: [https://www.babeljs.cn/blog/](https://www.babeljs.cn/blog/)

看了书和源码，发现研究 core-js 里的复用 polyfill 逻辑还是挺难理解的。但是 core-js 的源码很值得一读，想在工程化领域深耕的话 core-js 建议还是要多花时间研究一下！！今天先绕过去了。

一般我们不会直接在项目里用 core-js，core-js 的存在感挺低的，但是它被各种主流的工具、框架所使用。

### 最佳的polyfill方案

过时方案：es5-shim es6-shim

流行方案：

 `babel-polyfill`(融合了 `corejs` + `regenerator-runtime`) 结合 `@babel/preset-env` + `useBuiltins(entry)` + `preset-env targets`

```jsx
{
  "presets": [
    ["@babel/env": {
      useBuiltIns: 'entry',
      targets: {chrome: 44}
    }]
  ]
}
```

`@babel-preset-env` 定义了Babel需要的插件（理解为预设集），Babel根据 `preset-env` 设置的 `targets` 配置的支持环境**自动按需加载** polyfill

`useBuiltIns` 配置成 `usage`，可以真正根据代码情况分析AST并进行更细粒度的按需引用。

一个趋于完美的polyfill设计应该满足的核心原则是：按需打补丁，按需主要体现在两方面：

- 按照用户终端环境打补丁
- 按照业务代码使用情况打补丁