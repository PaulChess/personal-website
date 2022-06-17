---
title: 阿里云数据库 RDS Mysql 使用记录
date: 2022-06-17T00:00:00.000+00:00
lang: zh
duration: 8min
author: 沈佳棋
---

## 背景

由于最近团队内缺人，因此和另一位前端同事兼职用 `node` 搞一段时间后端开发。由于公司普遍是在内网开发，因此所有申请的公司资源都是在内网。然而我们在外网当自由人当习惯了，所以还是倾向于外网开发。但是数据库这个东西呢，一个人开发本地创建个 `localhost` 玩玩还行，两个人合作的话就很麻烦了，于是想着搞一台云数据库吧，刚好遇上阿里云新人 `¥9.9 1年`的优惠，还是挺香的。(毕竟平时最低配置大概是一百多块钱1个月)

<img src="/public/aliyun/0.png" />

下面是一些本机连云数据库的踩坑记录:  

## 1. 本地电脑如何连接阿里云数据库 RDS Mysql?

[基本文档](https://help.aliyun.com/document_detail/26138.html)

1. 进入 `实例列表` 查看数据库实例基本信息，点击 `查看连接` ⇒ `申请外网地址`(这里因为已经申请过了，所以显示的是 `释放外网地址`)，此时会得到外网地址和端口号

注释: 按文档描述，开放公网访问地址以及白名单设成 `0.0.0.0/0` 都是不安全的

<img src="/public/aliyun/1.png" />

2. 本地测试连接是否成功，进入命令行，输入(前提是本地已经装过Mysql并配置好环境变量):
```shell
# -h: host  -u: user -P: post -p: password
mysql -hrm-bp1ohuh0smjer8025vs.mysql.rds.aliyuncs.com -uatomroot -P3306 -p
```
此时发现会报错:

<img src="/public/aliyun/2.png" />

3. 无法解析 host, 说明 `dns解析` 没有配好。需要在网络下配置阿里云的 `公有dns`: `223.5.5.5` 和 `223.6.6.6`  
DNS配置文档: [https://www.alidns.com/knowledge?type=SETTING_DOCS#user](https://www.alidns.com/knowledge?type=SETTING_DOCS#user)  
下图是 Mac 的一个 WIFI 下配好的结果:

<img src="/public/aliyun/3.png" />

4. 再次输入上面的命令，输入密码后可以成功连上阿里云数据库
<img src="/public/aliyun/4.png" />

5. 数据库可视化操作工具 `DBeaver` 连接阿里云数据库如图，上面在命令行中能正常连通这里问题就不大。(关于 `DBeaver` / `Navicat` 以及 `Mysql` 等的安装本文不再赘述)
<img src="/public/aliyun/5.png" />

## 2. 云服务器 ECS 如何连接阿里云数据库 RDS Mysql?
待踩坑。(去年买的云服务器到期了 (灬ꈍ ꈍ灬)) 下次亲自试验后再补坑。

按官方文档，前提条件: ECS 和 RDS 的地域、网络类型要相同。

<img src="/public/aliyun/6.png" />

<img src="/public/aliyun/7.png" />

本文完。