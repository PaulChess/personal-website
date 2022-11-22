---
title: Yarn & CI环境下的npm优化及工程化问题解析
date: 2022-11-10T00:00:00.000+00:00
lang: zh
duration: 20min
---

## 一、Yarn简介

### 背景

Yarn出现在npm v3时期，为了解决npm的一些问题

Yarn解决了npm的哪些问题？

1. 安装包的确定性（一致性）问题 → yarn.lock 不管包的安装顺序如何，保障在不同的机型和环境上安装下来的包是一样的。（我猜是因为形成了确定的依赖关系图，包括实际安装的包版本都已经明确下来了）
2. **模块扁平安装方式**：假设不同的依赖包都依赖了`axios`这个包，但是安装的`axios` 版本是不一样的，如果不做处理的话会安装多个版本的 `axios`，很显然，这种做法是浪费资源的。yarn会采用一种策略把包的多个版本归并为一个版本。避免创建多个副本造成冗余。（问题：所以到底是什么策略呢？？）
3. 安装包速度慢的问题：采用请求排队的理念，类似于并发连接池，能够更好地利用网络资源。同时yarn还引入了重试机制。
4. 采用缓存机制，实现了离线模式。（类似于现在的npm cache?）问题：包是缓存在哪里呢？？

yarn.lock的结构：

```jsx
"@babel/cli@^7.1.6, @babel/cli@^7.5.5":
  version "7.8.4"
  resolved "http://npm.in.zhihu/@babel%2fcli/-/cli-7.8.4.tgz#505ffxxxx"
  integrity sha1-UF+xxxxxxxxx=
  dependencies:
	  commander "^4.0.1"
    convert-source-map "^1.1.0"
  optionalDependencies:
    chokidar "^2.1.8"
```

Yarn和npm的另一个显著区别是，`yarn.lock` 文件中的子依赖是不确定的，单独一个`yarn.lock` 确定不了 `node_modules` 的结构，还需要和 `package.json` 配合。

使用 `yarn` 的项目和使用 `npm` 的项目可以互转吗？

答案：`synp` 工具可以将 `yarn.lock` 和 `package.json` 进行互转

### 关于yarn的缓存

1. 可以通过 `yarn cache dir` 命令查看缓存目录，从而查看到缓存内容
    
    例如我自己电脑上的缓存目录是：`/Users/paulchess/Library/Caches/Yarn/v6`
    
2. `yarn` 使用的是 `prefer-online` 策略，也就是优先请求网络，如果请求不到再去使用缓存。

### yarn区别于npm的独有命令

yarn独有：

```bash
yarn import
yarn licenses
yarn pack
yarn why 
yarn autoclean
```

npm独有：

```bash
npm rebuild
```

### yarn安装机制

<img src="/public/npm1.jpeg" />

yarn的包安装过程

1. **检测包**

这一步主要检测项目中是否存在npm相关的文件，如 `package-lock.json` 如果存在，则给出提示，可能会引发冲突，同时在这一步会去解析OS、CPU等相关信息。

1. **解析包**

这一步重点就是分析依赖，得到一个依赖关系图，用的是Set数据结构来进行的存储。

首先先分析首层依赖，如`dependencies`、`devDependencies`、`optionalDependencies`

然后遍历首层依赖，获取每个依赖的包版本信息，再去递归解析子依赖，如果已经解析过，则将其放在队列中并将包状态标为 `已解析`。

在完成解析包这一步之后，就已经确定了所有依赖的具体版本信息和下载地址。

<img src="/public/npm2.jpeg" />

1. **请求包**

首先检查缓存中是否存在依赖包，如果不存在则走网络请求，现在到缓存目录

问题：

之前不是说yarn是 `prefer-online` 策略吗？这里是不是有点矛盾呢？

如何判断缓存中是否存在当前的依赖包？

Yarn会根据 `cacheFolder + slog + node_modules + [pkg.name](http://pkg.name)` 生成一个路径，判断系统中是否存在该路径判断该包是否有缓存。（按什么规则存就按什么规则找，这里的缓存路径规则其实就是一个协议）

这里其实解释了上面包缓存在哪里的问题。

**对于没有缓存的包，**Yarn会维护一个fetch请求队列，按照规则去请求对应的包。正常来讲应该都是 `[http://npm.in.zhihu/@babel%2fcli/-/cli-7.8.4.tgz#505ffxxxx](http://npm.in.zhihu/@babel%2fcli/-/cli-7.8.4.tgz#505ffxxxx)` 这种线上包地址，特殊情况是 `file` 协议开头的地址，代表是一个本地包（我猜应该是类似于npm link这种情况？），此时调用 `Fetch From Local` 从离线缓存中获取包（这里有个疑问啊：离线缓存和缓存的区别是啥？不是说没有缓存吗？那为什么能从离线缓存中去取呢？？）；否则调用 `Fetch From External` 来获取包。最终获取结果通过 `fs.createWriteStream` 写入缓存目录。

<img src="/public/npm3.jpeg" />

下载包流程

1. **链接包**

核心：生成`node_modules`目录

上一步将依赖包下载到缓存目录后，这一步开始遵循扁平化原则，将真正的包从缓存目录复制到项目的`node_modules`下。在复制依赖包之前，yarn会先解析 `peerDependencies` 内容，如果找不到匹配 `peerDependencies` 信息的包，则进行Warning提示，最终将依赖包复制到项目中。

问题：到底啥是peerDependencies啊？peerDependencies的作用是啥？到现在都没搞懂过。

1. **构建包**

依赖中如果存在二进制包，则需要对它进行编译，编译就是在这一步做的。

问题：这一步也太玄乎了，难道是生成了node_modules以后node_modules里有二进制包，将它编译完再写回去？

## 二、如何破解依赖管理困境

### 早期npm(v2)的问题？

早期npm是把 `node_modules` 设计成一个树形结构，很容易形成所谓 `依赖地狱` 的问题。

如何理解这个问题？

1. 依赖数层级太深
2. 重复安装依赖：浪费内存，安装过程慢，甚至会导致目录层级太深导致文件路径太长，最终导致在Windows上删除node_modules失败。

所以npm v3之后也把node_modules改成了扁平式结构。

**实际场景(基于npm v3)：**

1. `模块A v1.0` 依赖了 `模块B v1.0`，`A 1.0`、`B 1.0` 平铺
2. 此时加入了`模块C v1.0`，`模块C v1.0`依赖了`模块B v2.0`，由于A和C依赖的模块B版本大版本不同，版本要求不一致会引发冲突，也就是说 `B 2.0` 没办法平铺在 `node_modules` 中，此时 `npm v3版本` 会将 `B 2.0` 放到 `C 1.0` 目录下面。
3. 此时加入了`模块D v1.0`，`模块D v1.0` 也依赖了`模块B v2.0`，此时`B 2.0`也会被安装在`D 1.0` 目录下面。（这里我感觉可能会有依赖重复安装的问题）

思考：为什么是 `B v1.0` 出现在顶层而不是 `B v2.0` 出现在顶层呢？

答案：因为 `B v1.0` 是先被解析出来的，讲究个先来后到。（模块的安装顺序）

1. 此时加入了`模块E v1.0`，依赖了 `模块B v1.0`，那么它会直接用顶层的那个B模块。
2. 此时如果我们想将`模块A v1.0` 的版本更新为 `v2.0`，并且依赖 `模块B 2.0`，npm v3会怎么处理呢？
    - 删除模块A v1.0
    - 安装模块A v2.0
    - 留下模块B v1.0，因为模块E 1.0还在依赖它
    - 将模块B v2.0安装在模块A v2.0目录下，因为顶层已经有模块B v1.0了。

<img src="/public/npm4.jpeg" />

模块依赖关系

可以看到`模块B v2.0` 其实是重复安装的。所以说，**npm包的安装顺序对依赖树的影响很大**。模块安装顺序可能影响 `node_modules` 目录下的文件数量。

理想情况：

<img src="/public/npm5.jpeg" />

但是现实和理想还是有一定差距的。

方法1：我们可以把node_modules目录删掉，利用npm的依赖分析命令，得到一个更清爽的结构。

方法2：实际上，更优雅的方式是使用 `npm dedupe` 命令重新进行依赖分析。

而Yarn在安装依赖的时候会自动执行 `dedupe` 命令。整个优化安装的过程中遵循扁平化原则。

在整个依赖安装的过程中涉及到了缓存、系统文件路径、安装依赖树解析、安装结构算法等内容。

## 三、CI环境下的npm优化

CI环境一般指在容器环境里。

### 1. 合理使用 npm ci 命令和 npm install 命令

`npm ci` 就是为 CI环境准备的安装命令。

思考：`npm ci` 和 `npm install` 有什么不同呢？

答案：

- `npm ci` 会要求项目里必须有 `package-lock.json` 或者 `npm-shrinkwrap.json` 文件。
- `npm ci` 完全根据 `package-lock.json` 安装依赖，保障依赖统一；同时在安装过程中不需要求解依赖满足问题及构造依赖树（`直接跳过了解析阶段`），安装过程更迅速。
- `npm ci` 命令在执行安装时会先删除项目中原本的 `node_modules`，重新安装。
- `npm ci` 只能统一安装包，不能安装单个包。(比如 `npm ci axios` 这个就是不行的)。
- 如果 `package-lock.json` 文件和 `package.json` 文件冲突，那么执行 `npm ci` 的时候会直接报错。（问题：为什么要管 `package.json` 呢？什么情况下会引发冲突呢？）
- 执行 `npm ci` 命令永远不会改变 `package.json` 和 `package-lock.json` 文件的内容。（思考问题：那么 `npm install` 什么时候会改变它俩的文件内容呢？）

综上，在CI环境里使用 `npm ci` 要优于 `npm install`。特点：稳定、一致、迅速

### 2. 使用 package-lock.json 文件缩短依赖安装时间

`package-lock.json` 文件中缓存了每个包的具体`版本信息和下载链接`(依赖解析阶段)，不需要再去远程仓库进行查询即可直接进入`完整性校验环节`（问题：如何进行完整性校验？SHA加密？），减少了大量网络请求。

所以说前两步可以配合着一起使用。

### 3. 缓存node_modules

## 四、实际工程化问题解析

### 1. 为什么需要lockfiles，是否要将lockfiles提交到仓库？

`package-lock.json` 是 `npm v5` 版本引入的，作用是`锁定依赖安装结构`。保证在任意机器上执行`npm install` 得到的 `node_modules` 安装结果。

首先，为什么单一的 `package.json` 文件不能确定唯一的依赖树呢？

- 不同版本npm的安装依赖策略和算法不同，那么自然生成的依赖树也就不同。
- `npm install` 命令根据 `package.json` 文件中的 `semver-range version` 更新依赖，某些依赖可能远程仓库里的版本已经更新过了。（面试题：讲一讲 `semver-range`，`^`、`@` 等等这些都表示啥意思？？）

有了lockfiles，就可以保证项目依赖能够被精准还原。

其次，看一下 `package-lock.json` 每一个依赖下面的这些字段：

- **version** 版本号（确定的版本号）
- **resolved** 依赖包安装源（下载地址）
- **integrity** 表明包完整性的hash值
- **dev**：该模块是否为顶级模块的开发依赖。（问题：这个有啥用呢？？）
- **requires**：依赖包所有的依赖项。套娃
- **dependencies**：node_modules目录中的依赖包（特殊情况下才存在）（问题：啥意思？）— 只有子依赖依赖和当前已安装在根目录下的node_modules中的依赖产生冲突的时候才会有这个属性。涉及到上一节的嵌套依赖管理知识。

是否要提交lockfile到仓库，需要看项目定位：

1. 普通应用项目。建议提交。可以保证包安装的一致性。
2. 如果是开发一个供外部使用的库，需要谨慎考虑。因为库是要被其他项目依赖的，**如果不使用`package-lock.json`，就可以复用主项目中的依赖，减少内存占用，提高安装速度**。
3. 如果开发的库依赖了一个精准版本号的模块，提交lockfile可能会造成主项目同一个模块安装了不同版本号的情况。作为库开发者，如果真的有使用某个特定版本号依赖的需要，一个更好的方式是定义`peerDependencies`的内容。

推荐将 lockfiles 提交到仓库中，执行 `npm publish` 命令发布库的时候，是不会把 lockfiles 发布出去的。

关于lockfiles更加精细的处理建议：

1. 早期npm锁版本的方式是 `npm-shrinkwrap.json`，它和 `package-lock.json` 的不同之处就在于，在执行 `npm publish` 的时候是不会忽略它的，建议发布库之前把它过滤掉。（ps:现在npm版本这么新，这条建议已经算比较过时了）
2. 对 `package-lock.json` 文件的处理在 `npm v5.6` 以上版本才逐步稳定。`v5.0-v5.6` 中间有过几次更新。（建议 v5.6 以上，现在都 8.x了，所以这些历史过程中的问题看看就好）
    1. 在 `npm v5.0.x` 版本中，执行 `npm install` 命令时只会根据package-lock.json文件下载依赖，不管 `package.json` 里的内容是什么。也就是说并不会结合 `package-lock.json` 和 `package.json` 来进行依赖解析，比较单纯。
    2. 在 `v5.1.0 ~ v5.4.2` 版本之间，执行 `npm install` 将无视 package-lock.json，而下载最新的npm包并更新package-lock.json
3. `**npm v5.4.2` 之后需要注意如下事项（就是一些规则吧，这个比较重要）：**
    1. 如果项目中只有 `package.json`，会根据 `package.json` 生成 `package-lock.json`
    2. 如果同时存在 `package.json` 和 `package-lock.json`，同时 `package.json` 文件中的该包的 `semver-range` 和 `package-lock.json` 中的版本是兼容的，即使此时远程仓库中有新的适用版本，还是会优先根据 `package-lock.json` 下载。
    3. 如果同时存在，但是不兼容，那么 `package-lock.json` 文件会自动更新版本，与 `package.json` 中的 `semver-range` 兼版本兼容。
    4. 如果 `package-lock.json` 和 `shrinkwrap.json` 文件同时存在于项目根目录下，则 `package-lock.json` 文件。（优先级问题）

### 2. 为什么有xxxDependencies

有这样几种依赖声明：

1. **dependencies**
2. **devDependencies**
3. **peerDependencies**：同版本依赖
4. **bundledDependencies**：捆绑依赖
5. **optionalDependencies**：可选依赖

**关于 `dependencies` 和 `devDependencies` 关系的解释**

并不是只有 `dependencies` 下的模块才会被打包，而 `devDependencies` 下的模块一定不会被打包。取决于项目中是否实际使用了该模块。这两个声明在业务中更多起到的是规范作用。

**关于 `peerDependencies` 的解释**：

如果你安装我，那你最好也要安装我的依赖。例如，假设 `react-ui@1.2.2` 只提供一套基于 `React` 的UI组件库，它需要宿主环境（主项目）提供指定的React版本来搭配使用，此时就需要在 `react-ui` 的 `package.json` 中配置如下内容：

```bash
"peerDependencies": {
	 "react": "^17.0.0"
}
```

举例：对于插件类的项目，例如开发一个Koa中间件，这类插件脱离本体（Koa）是不能单独运行的，但是这类插件又不需要声明对本体的依赖，更好的方式是使用宿主环境的依赖，这就是 `peerDependencies` 主要的使用场景。特点是：

- 插件不能单独运行，插件运行的前提是，必须先安装核心依赖库
- 不建议重复下载核心依赖库，比如 `Koa`、`React` 等等
- 插件API设计必须符合核心依赖库的插件编写规范
- 在项目中，统一插件体系下的核心依赖库最好保持版本一致

**关于 `bundledDependencies` 的解释：**

表示捆绑依赖，和 `npm pack` 打包命令有关（面试题：讲一讲 `npm pack`?）。

假设有如下配置：

```json
{
  "name": "test",
  "version": "0.0.1",
  "dependencies": {
    "axios": "^19.0.0",
    ...
  },
  "devDependencies": {
    "request": "^18.0.0",
    ...
  },
  "bundledDependencies": [
    "bundledD1",
    "bundledD2"
  ]
}
```

在执行 `npm pack` 命令时，会产出一个 `test-0.0.1.tgz` 压缩包，压缩包中包含 `bundledD1` 和 `bundledD2` 两个安装包。业务方使用 `npm install test-0.0.1.tgz` 的时候会同时安装这两个包，前提是：这两个包必须在 `dependencies` 或 `devDependencies` 中声明过。

### 3. 版本规范：依赖库版本锁定行为

Vue官网上的内容：

> 每个Vue.js包的新版本发布时，一个相应版本的 `vue-template-compiler` 也会随之发布。编译器的版本必须和主包版本保持一致，这样 `vue-loader` 就会生成兼容运行时的代码。这意味着每次升级 vue的时候，也要相应升级vue-template-compiler
> 

作为库开发者，如何保证依赖包之间的最低版本要求》

对项目中的依赖进行比对和限制。例如 `create-react-app` 的核心 `react-script` 中，利用 `verifyPackageTree` 方法来进行比对和限制。

## 五、总结. npm正确的打开方式

1. 优先使用 `npm v5.4.2` 以上版本
2. 第一次搭建项目时候使用 `npm install <package>` 安装，把 `package-lock.json` 提交到仓库，但是不要提交 `node_modules`
3. 其他项目成员首次拉取代码后，执行 `npm install` 进行安装
4. **对于升级依赖包：**
    1. 依靠 `npm upgrade` 命令升级到新的小版本
    2. 依靠 `npm install <package-name>@<version>` 命令升级大版本
    3. 也可以直接修改 `package.json` 中的版本号，执行 `npm install` 来进行升级
    4. 本地验证升级后的新版本没问题，提交 `package-lock.json`
5. **对于降级依赖包：**
    1. 执行 `npm install <package-name>@<old-version>`，验证没问题后，提交 `package-lock.json`
6. **对于删除依赖包：**
    1. 执行 `npm uninstall <package>` 命令，后续同上
    2. 手动操作 `package.json` 中的包版本，后续同上
7. **任何团队成员提交 `package.json`、`package-lock.json` 文件更新后，其他成员应该拉取代码后执行 `npm install` 更新依赖（我认为非常重要！！）**
8. 任何时候都不要修改 `package-lock.json`
9. 如果 `package-lock.json` 文件出现冲突或问题，建议将本地的 `package-lock.json` 删掉，引入远程最新的 `package.json` 和 `package-lock.json` ，再执行 `npm install`