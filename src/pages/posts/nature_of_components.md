---
title: 组件的本质
date: 2022-02-16T00:00:00.000+00:00
lang: zh
duration: 8min
---

在组件之前比较流行的概念是 `模板引擎`，例如 EJS, JSP..  
举个例子 <ClientOnly><div class="i-ri-game-fill inline-block" text-lime-400></div></ClientOnly>：

```js
// EJS tempplate
const ejs = require('ejs')
peaple = ['jullie', 'tony', 'baby']
html = ejs.render('<%= peaple.join(", "); %>', { peaple })

document.getElementById('app').innerHTML = html
```

所谓模板引擎的概念: &nbsp;<u>字符串(模板) + 数据 ⇒ HTML</u>

而组件可以理解为一个函数，在给定模板的情况下，通过数据去渲染对应的 HTML 内容。  
当今的框架如 `Vue` 或 `React` 正是如此，这就是一个 `组件的本质`。  
区别仅在于当今的 MVVM 框架产出的内容并不是 HTML 字符串，而是 `Virtual DOM` 。

以 Vue 为例，一个组件最核心的内容就是 `render 函数`，而其他的 option 如 `data`、`computed`、`props` 等等都是为 render 函数提供数据来源。

<br />
<enhance-tag>render函数</enhance-tag> : 
<br />
render函数本可以直接产出 HTML 字符串，最终却产出了 `VNode (Virtual DOM)` ;  
VNode 最终还是要渲染真实的 DOM，这个过程通常叫做 `patch`;  
当 `数据变更` 时，组件会产出新的 VNode, 此时只需再次调用 patch 函数即可。

以下借助 [snabbdom](https://github.com/snabbdom/snabbdom) 的API来用代码描述这个公式：

```js
// example component
import { h, init } from 'snabbdom'

// init方法用来创建patch函数
const patch = init([])

// h函数用来创建vNode, 组件的产出就是vNode
const MyComponent = (props) => {
  return h('h1', props.title)
}

// 调用组件，产出第一个vNode
const prevVNode = MyComponent({ title: 'prev' })
// 将vNode渲染成真实DOM插入到页面中
patch(document.getElementById('app'), prevVNode)

// 数据变更，产出新的vNode
const nextVNode = MyComponent({ title: 'next' })
// 通过比对新旧VNode, 高效渲染真实DOM
patch(prevVNode, nextVNode)
```

<u>结论 ⇒ 组件的产出就是Virtual DOM, 渲染为真实DOM是用了patch。</u>  

采用 VNode 的优势：
1. 提高 DOM 操作性能
2. 带来分层设计，对渲染过程进行抽象，使得框架可以渲染到 Web(浏览器) 以外的平台，比如小程序，桌面端等
3. 实现 SSR

## 2. 组件的VNode如何表示?
首先要说明一点，VNode 是真实 DOM 的描述。  
想要把 VNode 渲染成真实 DOM，就需要用到渲染器(`Renderer`)。渲染器接收两个参数，分别是想要渲染的 <u>VNode</u> 以及 <u>元素挂载点</u>。  
  
举个例子 <div class="i-ri-game-fill inline-block" text-teal-400></div>：

```js
// vNode -> 描述真实dom
const vNodeElement = {
  tag: 'div',
}

// 准备一个渲染器
// 其原理根据vNode创建真实dom, 再将其插入到挂载点中
function render(vNode, container) {
  const el = document.createElement(vNode.tag)
  container.appendChild(el)
}

render(vNodeElement, document.getElementById('app'))
```

上述代码适用于标准的 HTML 标签，但是并不适用于组件。  
  
接下来需要探讨的是: 组件的 VNode 该如何表示?  
对于 HTML 标签的 VNode 来说，其 `tag属性` 的值就是标签的名字，对于组件，其 VNode 中的 tag 可以指向组件自身。想要正确渲染组件，还需要相应地修改  `render` 渲染器：

```js
// custom component: render函数返回vNode
class MyComponent {
  render() {
    return {
      tag: 'div',
    }
  }
}

// 描述组件vNode
const componentVNode = {
  tag: MyComponent,
}

// render渲染器
function render(vNode, container) {
  if (typeof vNode.tag === 'string')
    mountElement(vNode, container)
  else
    mountComponent(vNode, container)
}

// 挂载html元素
function mountElement(vNode, container) {
  const el = document.createElement(vNode.tag)
  container.appendChild(el)
}

// 挂载组件
function mountComponent(vNode, container) {
  // 创建组件实例
  // eslint-disable-next-line new-cap
  const instance = new VNode.tag()
  // 获取组件的渲染函数结果 => 即vNode
  instance.$vNode = instance.render()
  // 挂载
  mountElement(instance.$vNode, container)
}
```

结论就是: <u>可以让 VNode 的 tag 属性指向组件本身，从而使用 VNode 来描述组件。</u>

## 3. 组件的种类
描述组件的方式:

```js
// 第一种方式: 普通函数
function MyComponent(props) {}

// 第二种方式: 类
class MyComponent {}
```

这两种方式代表着两类组件, 分别是 `函数式组件`(Functional Component) 和 `有状态组件`(Stateful Component)。和 React 中的概念也很像，说明底层都是互通的。

区别如下：

函数式组件:

* 是一个纯函数
* 没有自身状态，只接收外部数据
* 产出vNode的方式：单纯的函数调用

有状态组件

- 是一个类，可实例化
- 可以有自身状态
- 产出 VNode 的方式：需要实例化，然后调用其 render 函数

本篇完