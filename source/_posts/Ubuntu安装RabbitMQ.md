---
title: Ubuntu安装RabbitMQ
date: 2018-05-07 22:33:52
tags: RabbitMQ
categories: AMQP
---
## 1. 安装

### 1.1 添加源 
``` bash
echo 'deb http://www.rabbitmq.com/debian/ testing main' | sudo tee /etc/apt/sources.list.d/rabbitmq.list
```
### 1.2 新增公钥（不加会有警告）
``` bash
wget -O- https://www.rabbitmq.com/rabbitmq-release-signing-key.asc | sudo apt-key add -
```
### 1.3 更新源
``` bash
sudo apt-get update
```
### 1.4 安装rabbitmq-server
``` bash
sudo apt-get install rabbitmq-server
```

## 2. 启动
### 2.1 启动服务
``` bash
/etc/init.d/rabbitmq-server start
```
> Usage: /etc/init.d/rabbitmq-server {start|stop|status|rotate-logs|restart|condrestart|try-restart|reload|force-reload}

### 2.2 查看服务
``` bash 
systemctl status rabbitmq-server
```

## 3. 配置
### 3.1 打开管理页面
``` bash
sudo rabbitmq-plugins enable rabbitmq_management
```
#### 3.1.1 查看安装的插件
``` bash
sudo rabbitmq-plugins list
```
#### 3.1.2 查看用户
``` bash
sudo rabbitmqctl list_users
```
### 3.2 新增管理员用户
``` bash
sudo rabbitmqctl add_user admin admin 
sudo rabbitmqctl set_user_tags admin administrator
```
### 3.3 使用新增的账户登陆管理页面
`http://127.0.0.1:15672`