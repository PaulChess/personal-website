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
  atomCalendar: {
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
  atomCalendar: {
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

OK. 变量名一一对应，之后在组件的源码里必然会通过某个方法来读取对应的变量值。例如：`this.t('atomCalendar.weekdays')`。 这一点后面再看。

接下来再看一下 `Locale` 组件本体，看看它的结构是怎样的？


