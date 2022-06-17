---
title: Vue-Cli postcss-preset-env 配置踩坑
date: 2021-11-03T00:00:00.000+00:00
lang: zh
duration: 5min
author: 沈佳棋
---

故事的背景是这样的：
前段日子根据公司的UI颜色规范(黑白板)整理了一套css变量提供给同事们使用，这样就不用每次写颜色的时候都要根据UI稿重复适配黑白板了，直接一个css变量搞定。提供的变量表如下:

```css
// cssVars.css
:root {
  --hxm-primary-1: #e93030;
  --hxm-primary-2: #f8c0c0;
  --hxm-primary-3: #f06f6f;
  ...
}
[theme-mode=black] {
  --hxm-primary-1: #fd4332;
  --hxm-primary-2: #662e2b;
  --hxm-primary-3: #a32222;
  ...
}
```

香是很香，但还是要考虑一下兼容性的：

<img src="/img1.webp" width="550"/>

在移动端的兼容性上，我们公司目前的要求是 `iOS >= 9`、`Android >= 4.4.4`，对比上面这张表，很可惜，就差了一丢丢就可以敞开使用了。

于是，推荐同事用css变量表的时候我跟他说，你可以配一个 `postcss-preset-env` 插件去做兼容。

## 配置postcss.config.js

上述同事的做法如下:

1.&nbsp;安装对应的开发依赖
```bash
"postcss": "^8.3.7",
"postcss-loader": "^6.1.1",
"postcss-preset-env": "^6.7.0"
```

2.&nbsp;在根目录下新建 `postcss.config.js`， 添加如下配置
```js
module.exports = {
  loader: 'postcss-loader',
  plugins: {
    'postcss-preset-env': {},
  },
}
```

3.&nbsp;在 `App.vue` 的 `style` 标签中全局引入css变量表文件，并进行使用测试
```html
<!-- App.vue -->
<template>
  <div id="app">
    <div class="test">
      Hello
    </div>
  </div>
</template>
```
```css
<style>
@import './assets/cssVars.css';

.test {
  color: var(--hxm-primary-1);
}
<style>
```

很可惜，没有生效。如何验证是否生效呢？

<img src="/img2.webp" width="420" />

如上图所示就是理想的结果，`postcss-preset-env` 会帮我们兼容兜底值，当浏览器支持css变量的时候就用css变量，不支持就使用兜底值，从而实现兼容。

然而，同事上述一顿操作的结果是:

<img src="/img3.webp" width="420" />

针对上述问题，我们找了很多的资料，看了很多博客上别人提出来的解决办法最终都没有效果，就在我感觉快要放弃挣扎的时候，翻了翻 vue-cli 仓库，找到了一条救命 [issue](https://github.com/vuejs/vue-cli/issues/4399)。

## 最终解决方法

提issue的这位老哥遇到了同样的问题。研究了一下他们的对话，发现终极原因是 `postcss-preset-env` 中少配了一个 `importFrom` 属性, 修改 `postcss.config.js` 配置如下:
```js
module.exports = {
  loader: 'postcss-loader',
  plugins: {
    'postcss-preset-env': {
      importFrom: './src/assets/cssVars.css',
    },
  },
}
```

问题奇迹般完美解决。 那么为什么要配置这个属性呢，作者给出的解释如下:
>The importFrom option specifies sources where variables like Custom Media, Custom Properties, Custom Selectors, and Environment Variables can be imported from, which might be CSS, JS, and JSON files, functions, and directly passed objects. Without this option the stylesheets in .vue files cannot access those globally defined varables.

大意就是，那些全局定义的 媒体查询、自定义属性、自定义选择、环境变量等等，不管是 css、 js、json、方法还是传递的对象都需要通过 `importFrom` 配置项具名导入进来。否则在 `.vue` 文件中使用的时候插件是找不到这些定义的内容的。

至此，问题就解决得差不多了，还有几个小的问题需要提醒一下：  
1.&nbsp;如果你的css变量不是定义在外部文件中，而是本身就定义在 .vue 文件内，那么是不需要配 importFrom 的。  
2.&nbsp;如果全局css变量分散在多个外部文件中怎么办呢？很幸运: importFrom 既支持字符串也支持数组，意味着你也可以这样写:
```bash
"postcss-preset-env": {
  'importFrom': [
    './src/assets/cssVars.css',
    './src/assets/cssVars.css'
  ]
}
```
3.&nbsp;注意外部css变量文件在 App.vue中通过 @import 导入，而不要在 main.js 里导入，那样也是无效的。

## 小结

上述问题确实是帮助同事一起排查了非常长的时间，搜索到的博客翻来翻去都差不多，所以决定自己写篇博客来记录一下这次踩坑，也能够帮助大家去排坑。
还有一点就是遇到问题多去看看对应仓库的issue，或许答案就在其中。平时也一定要注重积累，多去看看源码，原理性的东西，这样才不至于在排查问题的时候思维枯竭。
