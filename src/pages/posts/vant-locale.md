---
title: Vant 是如何做国际化的？
date: 2023-07-10T00:00:00.000+00:00
lang: zh
duration: 10min
author: 沈佳棋
---

在看到标题的时候，有些人可能会有疑问：印象里基础组件库的属性不都是用户传的么？用户传的是中文就展示中文；传的是英文就展示英文。因此组件的中英文展示应该由项目母体控制就行了吧？

我之前也一直是这么想的，直到有人问了我这个问题：下面两张图中画线的几块地方不需要传属性，但是 Demo 里可以自动切换中英文，是如何做到的？  

<div style="display:flex; justify-content: flex-start; align-items: center;">
  <img src="/public/atom-calendar-zh.png" style="width: 47%;" />
  <img src="/public/atom-calendar-en.png" style="width: 47%; margin-left: 10px;" />
</div>

以下是 Vant 的官方文档说明：

### Vant 的国际化配置：

#### 多语言切换

Vant 通过 `Locale` 组件实现多语言支持，使用 `Locale.use` 方法可以切换当前使用的语言。

```js
import { Locale } from 'vant'
// 引入英文语言包
import enUS from 'vant/es/locale/lang/en-US'

Locale.use('en-US', enUS)
```

这样就自动切换了英文，也就是从左图变换到右图。如果要切换到其他语言的话，从组件库中导入对应的语言包作为参数传给 `Locale` 组件即可。
  

#### 覆盖语言包

通过 `Locale.add` 方法可以实现文案的修改和扩展，示例如下：

```js
import { Locale } from 'vant'

const messages = {
  'zh-CN': {
    vanPicker: {
      confirm: '关闭', // 将'确认'修改为'关闭'
    },
  },
}

Locale.add(messages)
```

### Vant 的国际化原理

#### `Locale` 组件

显而易见，首要的核心就是这个 `Locale` 组件。得先理清楚它的结构。

顺着找到 vant 的 `src/locale` 目录，跟国际化相关的配置基本都是在这个目录下。结构如下：

```text
locale
└─ lang      # 语言包
   ├─ zh-CN.ts
   ├─ en-US.ts
   ├─ ja-JP.ts
   └─ .... 
└─ index.ts  # 入口文件
└─ README.md
└─ README.zh-CN.md
```

首先看一下语言包，挑 `zh-CN` 和 `en-US` 两个文件看一下，找到跟日历有关的配置：

**zh-CN.ts**:
```js
// zh-CN.ts
export default {
  // ...
  vanCalendar: {
    end: '结束',
    start: '开始',
    title: '日期选择',
    dateChooseEnd: '请选择结束时间',
    confirm: '确定',
    startEnd: '开始/结束',
    weekdays: ['日', '一', '二', '三', '四', '五', '六'],
    monthTitle: (year: number, month: number) => `${year}年${month}月`,
    rangePrompt: (maxRange: number) => `选择天数不能超过 ${maxRange} 天`,
  },
  // ...
}
```

**en-US.ts**:
```js
// en-US.ts
export default {
  // ...
  vanCalendar: {
    end: 'End',
    start: 'Start',
    title: 'Calendar',
    dateChooseEnd: 'Choose end date',
    startEnd: 'Start/End',
    weekdays: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'],
    monthTitle: (year: number, month: number) => `${year}/${month}`,
    rangePrompt: (maxRange: number) => `Choose no more than ${maxRange} days`,
  },
  // ...
}
```

OK. 变量名一一对应，之后在组件的源码里必然会通过某个方法来读取对应的变量值。例如：`this.t('vanCalendar.weekdays')`。 这一点后面再看。

接下来再看一下 `Locale` 组件本体，看看它的结构是怎样的？

```js
// locale/index.ts
export default {
  messages() {},

  use(lang: string, messages?: object) {},

  add(messages = {}) {},
}
```

显而易见，组件（与其说是组件，不如说是个对象）暴露了三个 API，`messages`、`use`、`add`，分别用于获取当前语言的配置对象、切换语言、覆盖语言。

关于这三个方法的远离其实就非常简单了，简单来说就是把当前语言以及当前语言包的配置挂在 Vue 的原型属性上，并将这两个属性使用 Vue utils 中的 `defineReactive` 方法变成响应式对象。然后这三个方法的核心就是针对原型上的这两个响应式变量进行相应的读写操作了。在 `add` 方法中的 `deepAssign` 方法倒是值得再细聊一下。可以单开一篇文章再剖析一下这个方法。  

以下为 `locale` 的源码：

```js
// locale/index.ts
import Vue from 'vue'
import { deepAssign } from '../utils/deep-assign'
import defaultMessages from './lang/zh-CN'

const proto = Vue.prototype
const { defineReactive } = (Vue as any).util

defineReactive(proto, '$vantLang', 'zh-CN')
defineReactive(proto, '$vantMessages', {
  'zh-CN': defaultMessages,
})

export default {
  messages() {
    return proto.$vantMessages[proto.$vantLang]
  },

  use(lang: string, messages?: object) {
    proto.$vantLang = lang
    this.add({ [lang]: messages })
  },

  add(messages = {}) {
    deepAssign(proto.$vantMessages, messages)
  },
}
```

注：`$vantMessages` 的格式示例：

```json
{
  "zh-CN": {
    "vanCalendar": {
     "weekdays": ["日", "一", "二", "三", "四", "五", "六"],
    }
  },
  "en-US": {
    "vanCalendar": {
      "weekdays": ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
    }
  },
}
```

### 组件调用国际化配置属性

看完了核心的 `Locale`，接下来要研究的就是组件是如何调用这些配置属性的？

首先看一下 Calendar 组件中如何调用 `weekdays` 这个属性：  
`t('weekdays')`。

重点自然就来到 `t` 这个函数的身上。当然，还有一个小问题，为什么不是调用 `t('vanCalendar.weekdays')` 而是可以直调 `t('weekdays')` 呢？

带着疑问来看一下 `t` 函数的源码：

```js
const [createComponent, bem, t] = createNamespace('calendar')
```

可以看出，`t` 是在调用完 `createNamespace` 这个方法之后返回的一个函数。

```js
// src/utils/create/index.ts
export function createNamespace(name: string): CreateNamespaceReturn {
  name = `van-${name}`
  return [createComponent(name), createBEM(name), createI18N(name)]
}
```

当传入组件名后，`name` 自动拼接了一个 `van` 前缀。

然后这里只需要关心 `createI18N` 这个函数。关于其他两个子函数也可以单独开一篇文章来讲。

#### createI18N

createI18N 方法的核心逻辑也是比较简单的，先贴一下源码：

```js
import { get, isFunction } from '..'
import { camelize } from '../format/string'
import locale from '../../locale'

export function createI18N(name: string) {
  const prefix = `${camelize(name)}.`

  return function(path: string, ...args: any[]): string {
    const messages = locale.messages()
    const message = get(messages, prefix + path) || get(messages, path)

    return isFunction(message) ? message(...args) : message
  }
}
```

首先，这一行代码可以解释我们先前的一个小疑惑：

```js
const prefix = `${camelize(name)}.`
```

`camelize` 方法是把 string 转成驼峰。这行代码解释了为什么我们在用 `t` 函数调用的时候参数中不需要使用 `vanCalendar.`，因为这里自动处理掉了这个前缀。

接下来，这个方法返回了一个子方法，其实就是我们外部调用的所谓 `t` 函数。它接收一个 `path`，即国际化配置项中的参数。这个子方法中调用了 `Locale` 对象中的 `messages` 方法，它会根据当前的语言获取对应的语言对象，得到的结果示例如下：

```json
{
  "vanCalendar": {
    "weekdays": ["日", "一", "二", "三", "四", "五", "六"],
  }
}
```

然后 `get` 方法就是根据 `key` 来获取对象的属性值了，这个方法后面也可以详细分析一下它有何特别之处。这里还有个细节值得注意一下：

```js
const message = get(messages, prefix + path) || get(messages, path)
```

这行代码其实做了一个兼容，即我们既可以使用 `t('weekdays')`，也可以使用 `t('vanCalendar.weekdays')`，这两种写法都可以。

最后理论上就返回对应的值了。但是 vant 还有一个细节：

```js
return isFunction(message) ? message(...args) : message
```

这里还判断了值是不是方法，如果是方法的话会调用这个方法返回真正的结果值。从 `lang/zh-CN.ts` 文件中还可以找到 `rangePrompt: (maxRange: number) => `Choose no more than ${maxRange} days`,` 这样的配置，所以在配置多语言参数值的时候也是可以传方法的。

### 结语

本文详细（啰嗦）阐述了 vant 的国际化实现方式；

除了 `vant` 源码中的 `Locale` 之外，还有一处地方跟国际化相关，即 vant 文档中的国际化切换，那里是封装了一个 `mixin`，当然，在 `vant3` / `vant4` 中就是一个 vue hook。关于那块切换后面也可以写一篇文章详细梳理一下。

本文完。