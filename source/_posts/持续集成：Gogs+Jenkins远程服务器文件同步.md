---
title: 持续集成：Gogs+Jenkins远程服务器文件同步
date: 2018-07-06 11:48:39
tags: [Gogs, Jenkins, CI]
categories: 持续集成CI
---

## 1. Gogs配置

### 1.1 仓库设置（仓库管理员）

![Git仓库](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB182IxkyAnBKNjSZFvXXaTKXXa)

### 1.2 添加Web钩子

![web钩子](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1Rc.dDeySBuNjy1zdXXXPxFXa)

### 1.3 Web钩子配置

![webhook配置](http://g-search1.alicdn.com/img/bao/uploaded/i4/TB1EVl9kAZmBKNjSZPiXXXFNVXa)

> 推送地址：http://[jenkins-server]/gogs-webhook/?job=[jenkins的job名称]
>
> 密钥文本：jenkins的job需要配置密钥才能连接

## 2. Jenkins配置

### 2.1 安装插件

- [Gogs plugin](https://wiki.jenkins-ci.org/display/JENKINS/Gogs+Webhook+Plugin)
- [Publish Over SSH](http://wiki.jenkins-ci.org/display/JENKINS/Publish+Over+SSH+Plugin)

> 系统管理->插件管理->可选插件，直接搜索，点击“直接安装”即可。

### 2.2 SSH设置

![ssh设置](http://g-search1.alicdn.com/img/bao/uploaded/i4/TB1iOXukScqBKNjSZFgXXX_kXXa)

> 系统管理->系统设置

### 2.3 新建Job

![新建job](http://g-search2.alicdn.com/img/bao/uploaded/i4/TB1gam_kCMmBKNjSZTEXXasKpXa)

> 填写job名称，选择“构建一个自由风格的软件项目”

### 2.4 Job配置

- General

![General](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1FcJrDxWYBuNjy1zkXXXGGpXa)

> 项目名称即为job名称，如需更改，请保持gogs web钩子的推送地址中job与此一致

- Gogs Webhook

![Gogs Webhook](http://g-search1.alicdn.com/img/bao/uploaded/i4/TB1n0kYkpkoBKNjSZFEXXbrEVXa)

> 如果gogs web钩子设置了密钥，就需要勾选“Use Gogs secret”，并填写Secret

- 源码管理

![源码管理](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1f9SdksIrBKNjSZK9XXagoVXa)

> 点击“Add”添加账号密码信息，填写Username和Password

- 构建触发器

![构建触发器](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1d8vpktcnBKNjSZR0XXcFqFXa)

- 构建

  **选择Send files or execute commands over SSH**

![构建](http://g-search1.alicdn.com/img/bao/uploaded/i4/TB1uAOYDXmWBuNjSspdXXbugXXa)

> “*****”表示复制当前仓库下所有文件

附脚本（复制，同名则备份）：

```sh
#!/bin/sh

upload_dir=/home/test/

www_dir=/home/test1/

backupexist()

	{

		filelist=`ls $1`

		for file in $filelist

		do

			if test -f $1$file 

			then

				if test -f $2$file 

				then

					cp $2$file $2$file"."`date +%Y-%m-%d`".del"

				fi

			else

				backupexist $1$file"/" $2$file"/"

			fi

		done

	}

	backupexist $upload_dir $www_dir

	cp -R $upload_dir"." $www_dir

```

## 3. 测试

### 1. 触发 Web 钩子

- 测试推送

![测试推送](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1SIuhaNtnkeRjSZSgXXXAuXXa)

- git push命令触发

### 2. Jenkins Job状态

![Jenkins Job状态](http://g-search3.alicdn.com/img/bao/uploaded/i4/TB1_npzDxWYBuNjy1zkXXXGGpXa)

> - 蓝色：任务最近一次构建是成功的
> - 红色：任务最后一次构建是失败的
> - 黄色：任务最后一次构建表示成功了，但不稳定（主要是因为有失败的测试）
> - 灰色：任务从未被执行过或被禁用了

