---
title: Redis 学习笔记
date: 2019-03-05 09:04:04
tags: [Linux, Redis]
categories: 白科技
---
本文是关于 Redis 的学习笔记
<!--more-->
## Redis 安装

采用 Docker 方式

```shell
docker pull redis
docker run --name myredis -d -p6379:6379 redis
docker exec -it myredis redis-cli
```

## Redis 基础数据结构

Redis共有5种基础数据结构，分别为：string (字符串)、list (列表)、set (集合)、hash (哈希) 和 zset (有序集合)。Redis所有的数据都是以键值对存在。

### string (字符串)

- 键值对

```shell
> set name Andy
ok
> get name
"Andy"
> exists name
(integer) 1
> del name
(integer) 1
> get name
(nil)
```

- 批量键值对

```shell
> set name1 andy
OK
> set name2 bob
OK
> mget name1 name2 # 批量获取
1) "andy"
2) "bob"
> mset name1 ANDY name2 BOB name3 CINDY  # 批量更改，可添加
OK
> mget name1 name2 name3
1) "ANDY"
2) "BOB"
3) "CINDY"
```

- 过期与 set 命令扩展

`expire` 这个功能常用来控制缓存的失效时间，`setex` 用来创建数据并控制缓存失效时间，`setnx` 创建不存在的数据。

```shell
> expire name1 1 # 1s后失效name1
(integer) 1
> expire name2 2 # 2s后失效name2
(integer) 1
> keys *
(empty list or set)
> setex name 10 cindy # 创建 name-cindy 并在10s后失效
OK
> setnx name cindy # 10s内创建 name-cindy，未成功
(integer) 0
> setnx name cindy # 10s后创建 name-cindy，成功
(integer) 1
> get name
"cindy"
```

- 计数

如果 value 是一个整数，可以进行自增操作。

```shell
> set age 10
OK
> INCR age
(integer) 11
> INCRBY age 5
(integer) 16
> INCRBY age -7
(integer) 9
```

### list (列表)

### set (集合)

### hash (哈希)

### zset (有序集合)
