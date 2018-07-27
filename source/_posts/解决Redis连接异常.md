---
title: 解决Redis连接异常
date: 2018-06-07 16:25:08
tags: [ Redis, Spring boot]
categories: BUG
---

重新部署项目后，后台服务只要用到了Redis的全都报错，不知道哪里改动了。

**报错代码**：

```java
redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
...
redis.clients.jedis.exceptions.JedisDataException: ERR Client sent AUTH, but no password is set
```



解决办法：

1. 进入redis容器(或服务器上)

```bash
docker exec -it my-redis /bin/bash
```

2. 设置密码

```
$ redis-cli
127.0.0.1:6379> CONFIG SET requirepass "123456"
```

3. 修改服务配置

```
spring.redis.password=123456
```

4. 重启后台服务

