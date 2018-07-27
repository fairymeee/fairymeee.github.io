---
title: Ubuntu小技巧
date: 2018-05-07 21:41:47
tags: [Linux, Ubuntu]
categories: Ubuntu
---
## 1. 解决SSH超时断开连接
-  在客户端电脑上编辑（**需要root权限**）`/etc/ssh/ssh_config`，并添加如下一行：

       ServerAliveInterval 60

此后该系统里的用户连接SSH时，每60秒会发一个KeepAlive请求，避免被踢。

## 2.解决`/boot`分区空间不足
- 查看系统已安装的内核
```bash
dpkg --get-selections|grep linux-image
```
- 查看当前使用的内核
```bash
uname -a
```
- 删除旧的内核
```bash
sudo apt-get autoremove linux-image-4.13.0-36-generic linux-image-4.13.0-38-generic
```
> 可保留最近的两个内核版本，其它可全部删除

- 查看分区使用情况
```bash
df -lh
```