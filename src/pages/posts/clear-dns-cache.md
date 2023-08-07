---
title: MacOS13 清除 DNS 缓存
date: 2023-08-07T00:00:00.000+00:00
lang: zh
duration: 2min
author: 沈佳棋
---

### 问题背景

* 电脑系统：MacOS 13 Ventura
* 连接网络：公司办公内网
* 问题现象：公司内部分系统网页地址打不开，提示错误：`Err_Empty_Response`
* 附加信息：无论是否打开代理软件及 `Proxy SwitchyOmega` 浏览器插件，都无法访问

### 解决方案

清除 DNS 缓存。  
不同的系统清除 DNS 缓存的命名不同，这里我贴一下 `MacOS 13 Ventura` 的解决方案：

```bash
sudo killall -HUP mDNSResponder
```

输入开机密码，如果没有任何提示，代表清除 DNS 缓存成功。  


### 结果

清除 DNS 缓存后，关闭代理软件及 `Proxy SwitchyOmega` 插件，可以正常访问网站。  

至于其中的原理，DNS 缓存在这个过程中的影响，值得深入研究一下。