---
title: NodeJS整合CAS
date: 2018-05-23 11:16:20
tags: [ nodejs, cas]
categories: CAS
---

# 1 背景

子系统使用NodeJS实现，现需把该子系统整合到已实现的CAS系统中，实现统一控制访问。本文只讲述实现过程中所涉及到的内容，其它内容不做深入研究。

基于开源项目：

```
https://github.com/TencentWSRD/connect-cas2
```

# 2 实现

## 2.1 安装

```bash
npm install connect-cas3 --save
```

> 这里使用的connect-cas3，也可以直接connect-cas2，可能会有细微的差别。

## 2.2 涉及特性

1. 非代理模型下的CAS协议的登入、登出
2. 单点登出

## 2.3 快速开始

注意：

1. 务必在使用casClient.core()中间件之前初始化session
2. 如果需要启用单点登出，并且使用了bodyParser，那么必须在bodyParser之前使用casClient中间件。 因为CAS Client接收单点登出的请求需要拿到一个POST请求的RAW body，而在bodyParser之后并没有办法办到这个事情，因为bodyParser已经把请求拦截了。

```javascript
var express = require('express');
var ConnectCas = require('connect-cas3');
var bodyParser = require('body-parser');
var session = require('express-session');
var cookieParser = require('cookie-parser');
var MemoryStore = require('session-memory-store')(session);

var app = express();
var router = express.Router();

app.use(cookie_parser());
app.use(session({
      secret            : 'a long secret',//自定义
      name              : 'xxxxxxx.sid',//xxxx可替换
      resave            : true,
      saveUninitialized : false,
      store: new MemoryStore()
    }));

var casClient = new ConnectCas({
    debug: true,
    ignore: [],
    restletIntegration: null,
    match: [],
    servicePrefix: 'http://localhost:3000',
    serverPath: 'https://cas-server:8443/cas',
    paths: {
        validate: '/validate',
        serviceValidate: '/serviceValidate',
        proxy: '',
        login: '/login',
        logout: '/logout',
        proxyCallback: ''
    },
    redirect: true,
    gateway: false,
    renew: false,
    ssoff: true,// 单点登出配置
    slo: true,// connect-cas3内的单点登出开关，但没用上
    cache: {
        enable: false,
        ttl: 3 * 60 * 1000,
        filter: []
    },
    fromAjax: {
        header: 'x-client-ajax',
        status: 418
    }
});
app.use(casClient.core());

// 登入
app.get(/\/.*/, function (req, res, next) {
    if (!req.session.cas.user) {
      return next();
    }

    const username = req.session.cas.user;
    req.session.loggedIn = true;
    req.session.username = username;
    // ...
    return next();
});

// 手动登出
router.get('/logout', casClient.logout());

// 单点登出，cas服务端发出登出请求
// 如果options.ssoff=false，则result=undefined
router.post('/logout', function (req, res, next) {
    if (!req.body.logoutRequest) {
        next();
        return;
    }
    var st = req.body.logoutRequest.match(/<samlp:SessionIndex>(.*)<\/samlp:SessionIndex>/)[1];
    req.sessionStore.get(st, function (err, result) {
        if (err) {
            return;
        }
        if (result && result.sid) req.sessionStore.destroy(result.sid);
        req.sessionStore.destroy(st);
    });
});

app.use(body_parser.json());
app.use(body_parser.urlencoded({ extended : true }));
```

## 2.4 其它配置

请参考[connect-cas2](https://github.com/TencentWSRD/connect-cas2/blob/master/README.zh.md)配置文档

# 3 总结

## 3.1 登入

匹配所有路径，实现登入中间件，得到cas返回的user等信息，并处理系统登入后的业务。

## 3.2 登出

### 3.2.1 手动登出

使用自带的登出`casClient.logout()`,下附源码：

```javascript
ConnectCas.prototype.logout = function() {
  var options = this.options;

  return function(req, res, next) {
    if (!req.session) {
      return res.redirect('/');
    }
    // Forget our own login session

    if (req.session.destroy) {
      req.session.destroy();
    } else {
      // Cookie-based sessions have no destroy()
      req.session = null;
    }

    // Send the user to the official campus-wide logout URL
    // utils.getPath: options.serverPath + options.paths.logout + '?service=' + encodeURIComponent(options.servicePrefix + options.paths.validate);
    return res.redirect(utils.getPath('logout', options));
  };
};
```

### 3.2.2 单点登出

单点登出没使用自带的，源码写的应该有个拦截器，监听登出请求。

登出逻辑：

1. 单独写一个登出`/logout`路由,并注册到CAS服务端
2. 拿到CAS服务端发过来的`logoutRequest`并从中得到st票据
3. 销毁st票据及session