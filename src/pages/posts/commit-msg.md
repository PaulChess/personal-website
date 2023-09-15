---
title: commit 信息前缀
date: 2023-09-15T00:00:00.000+00:00
lang: zh
duration: 2min
author: 沈佳棋
---

在提交 commit 信息的时候我时常会纠结我的 commit 内容属于是什么类型的，除了 `feat`、`chore`、`fix` 这些常用的之外也会自己定义一些例如 `docs`、`styles` 等等。  
  
无意间看到这样一个 VSCode 插件，旨在提交 commit 信息的时候选择并自动插入提供好的 emoji 表情之一。[GitHub 地址](https://github.com/maixiaojie/git-emoji-zh)。

<img src="https://github.com/maixiaojie/git-emoji-zh/raw/master/images/features.gif" />

`src/api/git_emoji_zh.ts` 中的 emojis 列表可能能为以后分类 commit 信息作一定的参考：

```ts
[
  {
    emoji: '🎉',
    entity: '&#x1f3a8;',
    code: ':tada:',
    description: '初次提交/初始化项目😬',
    name: '庆祝',
  },
  {
    emoji: '🤔',
    code: ':ideas:',
    description: '思考 & 计划🥺',
    name: '思考',
  },
  {
    emoji: '✨',
    entity: '&#x1f525;',
    code: ':fire:',
    description: '引入新功能🙃',
    name: '火花',
  },
  {
    emoji: '💄',
    entity: '&#x1f525;',
    code: ':lipstick:',
    description: '更新 UI 和样式文件',
    name: '口红',
  },
  {
    emoji: '🐛',
    entity: '&#x1f41b;',
    code: ':bug:',
    description: '修复 bug😭',
    name: 'bug',
  },
  {
    emoji: '🚑',
    entity: '&#128657;',
    code: ':ambulance:',
    description: '添加重要补丁😔',
    name: '急救车',
  },
  {
    emoji: '🎨',
    entity: '&#x2728;',
    code: ':sparkles:',
    description: '改进代码结构/代码格式😍',
    name: '调色板',
  },
  {
    emoji: '📦',
    entity: '&#x1f4dd;',
    code: ':pencil:',
    description: '添加新文件/引入新功能😋',
    name: '添加',
  },
  {
    emoji: '✅',
    entity: '&#x1f680;',
    code: ':rocket:',
    description: '增加测试代码🤑',
    name: '测试',
  },
  {
    emoji: '📖',
    entity: '&#ff99cc;',
    code: ':lipstick:',
    description: '添加/更新文档😁',
    name: '文档',
  },
  {
    emoji: '🚀',
    entity: '&#127881;',
    code: ':tada:',
    description: '发布新版本😄',
    name: '发布',
  },
  {
    emoji: '👌',
    entity: '&#x2705;',
    code: ':white_check_mark:',
    description: '提高性能/优化🤪',
    name: '优化',
  },
  {
    emoji: '🔖',
    entity: '&#x1f516;',
    code: ':bookmark:',
    description: '发布版本/添加标签😃',
    name: '书签',
  },
  {
    emoji: '🔧',
    entity: '&#x1f527;',
    code: ':wrench:',
    description: '修改配置文件🙄',
    name: '配置',
  },
  {
    emoji: '🌐',
    entity: '&#127760;',
    code: ':globe_with_meridians:🤒',
    description: '多语言/国际化',
    name: '国际化',
  },
  {
    emoji: '💡',
    entity: '&#128161;',
    code: ':bulb:',
    description: '文档源代码😶',
    name: '灯泡',
  },
  {
    emoji: '🍻',
    entity: '&#x1f37b;',
    code: ':beers:',
    description: '像喝多了写的代码😳',
    name: '啤酒',
  },
  {
    emoji: '🥚',
    entity: '&#129370;',
    code: ':egg:',
    description: '添加一个彩蛋🤓',
    name: '蛋',
  },
  {
    emoji: '🙈',
    entity: '&#8bdfe7;',
    code: ':see_no_evil:',
    description: '添加或修改.gitignore文件😯',
    name: '不可见',
  },
  {
    emoji: '🏷️',
    entity: '&#127991;',
    code: ':label:',
    description: '添加或更新types(Flow, TypeScript)🤨',
    name: '标记',
  },
]
```

如果后续有看到新的与 git commit 规范相关的资料会放在这篇文章的参考资料里。  
这个插件的源码有时间也可以研究一下。