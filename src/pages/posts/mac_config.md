---
title: 一名前端程序员的Mac新机配置
date: 2022-08-29T00:00:00.000+00:00
lang: zh
duration: 18min
author: 沈佳棋
---

### 浏览器
**Google**：在可以翻墙的情况下登一下账号，账号上绑定的一些插件、收藏、设置等等都会同步过来，还是挺方便的。

### 翻墙工具
**[ByWave](https://bywa.cc/)**：下载完登一下账号就可以用了  

### 即时通信软件
1. **同花顺 vanish** (纯纯社畜实锤了，还好，历史记录会自动同步过来)
2. **微信**

### 输入法
**搜狗拼音**  
在 mac 上使用搜狗拼音其输入习惯是贴近于 windows 的，比如 `shift` 切换中英文，`中/英` 键切换大小写。
绑定账号的话之前的皮肤设置、输入联想等等也都能回来 (〃'▽'〃)

### 系统默认设置修改
1. 触控板：
选中轻点点按：单指轻按单击，双指轻按右击
2. Lauchpad 图标大小修改
3. [Mac 自定义文件夹图标](https://cn.sync-computers.com/spruce-up-your-desktop-with-custom-folder-icons-macos)

### 云笔记
**notion** 下载 mac 客户端 (绝对的笔记📒之王)

### 音乐
1. **QQ 音乐**
2. **网易云音乐**

### 邮箱
1. **foxmail**

### 脑图工具
1. **XMind**

### 设计软件
1. **Figma**
2. **Sketch**

### 音视频剪辑
1. 音频软件：**Logic Pro**
2. 视频软件：**Final Cut Pro**

## 程序员👨🏻‍💻必装：
### Item2 + oh-my-zsh
非常好用的终端

[oh-my-zsh](https://ohmyz.sh/) 的作用是对其进行颜值美化  
oh-my-zsh 基本配置可以参考这篇文章：[文章地址](https://zhuanlan.zhihu.com/p/290737828)

我安装的插件：
- **zsh-autosuggestions**
- **autojump**
- **zsh-syntax-highlighting**

配色：
- **Tango Dark**

主题：
- **powerlevel9k/powerlevel9k**

配置效果：
<img src="/public/img3.png" />

### HomeBrew
Homebrew 是一款自由及开放源代码的软件包管理系统，用以简化 macOS 和 linux 系统上的软件安装过程。  
使用国内镜像源进行安装:  
```bash
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

homebrew 的目录：
```bash
cd "$(brew --repo)"
```

### Visual Studio Code
关于 settings 和插件的同步，用 `Settings Sync` 这个插件   
基本上可以参考这篇文章： [文章地址](https://www.cnblogs.com/lychee/p/11214032.html)  
  
配置过程中有个大坑文章里没提到，这里补充一下：就是在输入 `> sync advanced` 选了之后需要在 `syncLocalSettings` 文件的 `ignoreUploadFolders` 配置项中加上一些需要过滤的文件夹 `History`、`GlobalStorage`:  [Issue地址](https://github.com/shanalikhan/code-settings-sync/issues/1341#issuecomment-1094088898)

### Node
版本: **16.17.0LTS**，因为项目里用到 `bit`， `bit` 对 node 版本的要求较高。

```bash
# 安装 node16
brew install node@16

# If you need to have node@16 first in your PATH, run:
echo 'export PATH="/opt/homebrew/opt/node@16/bin:$PATH"' >> ~/.zshrc

# For compilers to find node@16 you may need to set:
echo 'export LDFLAGS="-L/opt/homebrew/opt/node@16/lib"' >> ~/.zshrc
echo export CPPFLAGS="-I/opt/homebrew/opt/node@16/include" >> ~/.zshrc

source ~/.zshrc

# 查看版本
node -v
# 输出 v16.17.0
npm -v
# 输出 8.15.0
```

### nvm、nrm
装完 node，就不得不装与其配套且与 npm 容易混淆的两大工具。
- **nvm**：`node` 版本管理器，也就是说：一个 `nvm` 可以管理多个 `node` 版本（包含 `npm` 与 `npx`），可以方便快捷的 `安装`、`切换` 不同版本的 `node`。

```bash
# 安装 nvm
brew install nvm

# 创建 .nvm
mkdir ~/.nvm

# 环境变量
echo '"export NVM_DIR="$HOME/.nvm"
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"  # This loads nvm
  [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion"' >> ~/.zshrc

source ~/.zshrc

# 查看版本号
nvm -v
# 输出 0.39.1  表明安装成功

# 使用 nvm
# 查看可用的 node 版本
nvm ls
# 安装 node
nvm install v18.8.0
nvm install lts/fermium
# 使用 node
nvm use v18.8

# nvm 设置默认node版本，不然打开新的tab的时候node版本又切到默认版本了
nvm alias default v16.17.0
```

- **nrm** 帮助你快速新增、切换 npm 镜像源  

这是个 npm 包，所以直接看文档即可：[https://www.npmjs.com/package/nrm](https://www.npmjs.com/package/nrm)


### 包管理器三剑客：npm、yarn、pnpm
- **npm**
- **yarn**
- **pnpm**

`@antfu/ni` 无脑管理上面三个包管理器


### 其他一些全局包
- **Typescript**
- **rimraf**
- **bit (node16.16.0+) 安装：npx teambit/bvm install**
- **whistle** (抓包工具)，配合浏览器 `Proxy SwitchyOmega` 插件一起使用
- **nodemon**
- **@vue/cli**
- ……  (自行 npm install -g xxx 即可)


### 接口调试工具
- **postman**
- **ApiFox （一个大而全的工具）**


### 数据库
1. Mysql 安装配置  root：[文章地址](https://juejin.cn/post/6844903831298375693)

```bash
brew install mysql

brew services start mysql

mysql_secure_installation

# 设置密码
```
2. MySQLWorkbench/DBeaver 客户端安装配置


### 管理 host 的工具
**SwitchHosts**


### 配置GitHub SSH
参考这篇文章： [文章地址](https://www.cnblogs.com/ayseeing/p/3572582.html)


    
## Tips
1. 通过 git clone 下载报错：Failed to connect to [raw.githubusercontent.com](http://raw.githubusercontent.com/) port 443 after 2 ms: Connection refused

解决办法，配置 GitHub host:
```bash
199.232.68.133 raw.githubusercontent.com
199.232.68.133 user-images.githubusercontent.com
199.232.68.133 avatars2.githubusercontent.com
199.232.68.133 avatars1.githubusercontent.com
```

2. 通过 brew install 下载报错：fatal: not in a git directory  
解决办法，输入 `brew -v` ，会给出提示，照做就行。

1. mac 调整 LauchPad 图标大小： [文章地址](https://www.jianshu.com/p/60315dfcda53)
```bash
# 调整为一排7个
defaults write com.apple.dock springboard-rows -int 7
# 重置 LauchPad
defaults write com.apple.dock ResetLaunchPad -bool TRUE;killall Dock
```

4. vscode 浏览器下载特别慢  
解决办法：[用镜像](https://zhuanlan.zhihu.com/p/112215618)

5. mac npm安装全局依赖，失效找不到`commad is not found`: [文章地址](https://blog.csdn.net/yunchong_zhao/article/details/121214241)

6. mysql 数据库：解决Node.js mysql客户端不支持认证协议引发的“ER_NOT_SUPPORTED_AUTH_MODE”问题：[文章地址](https://waylau.com/node.js-mysql-client-does-not-support-authentication-protocol/)

7. node 启动项目报错：error:0308010C:digital envelope routines::unsupported：[文章地址](https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported)

一般是因为本地node版本与项目不匹配，如果本地版本比较高，node17+可能会出现这样的问题，

建议降低到 node16