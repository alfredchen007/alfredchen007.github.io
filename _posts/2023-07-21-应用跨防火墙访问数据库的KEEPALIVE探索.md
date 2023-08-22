---
layout: post

title: "应用跨防火墙访问数据库的KEEPALIVE探索"

date: 2023-07-21 

description:  

cover: 'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-08-22-%E6%88%AA%E5%B1%8F2023-08-22%2015.43.01.png'

color: rgb(222,315,333)

tags: [linux]
---

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-08-22-%E6%88%AA%E5%B1%8F2023-08-22%2015.43.01.png)

### 问题背景

应用端使用 druid 连接池访问数据库，中间有一层山石的防火墙，防火墙上配有空闲 session 超时的策略， 并且触发该配置时不会进行四次挥手，导致应用端连接池逐渐满了无法释放最后抛出异常。

### 配置 TCP KEEPALIVE 的几处地方

#### 1、在应用端驱动层面做

这种方式需要数据库提供的驱动支持 tcp keepalive，例如 TeraData 数据库 jdbc 驱动的文档里有明确写支持KEEPALIVE，需要连接时声明该参数

#### 2、在应用端连接池层面做

例如 druid 连接池在 Druid-1.0.27 之前的版本，建议使用 TestWhileIdle 来保证连接的有效性，在 1.0.28之后的版本加入 KeepAlive 配置，缺省关闭。

打开 KeepAlive 后

1. 初始化连接池时会填充到 minIdle 的数量
2. 连接池中的 minIdle 数量以内的连接，空闲时间超过 minEvictableIdleTimeMillis，会执行 KeepAlive 操作
3. 网络断开等原因产生的死链接，被 ExceptionSorter 检测出来后清除，自动补充连接到 minIdle 数量。

所以应用端 druid 开启 KeepAlive 后（ testWihleIdle 为 true、testOnBorrow 为 false ），按照 timeBetweenEvictionRunsMillis 间隔进行有效性验证，实现连接保活。当空闲时间大于**minEvictableIdleTimeMillis** 时断开空闲连接，直到连接池中的连接数到 minIdle 为止。

在这个层面进行配置需要 timeBetweenEvictionRunsMillis 时间小于防火墙定义的空闲超时时间。

#### 3、在服务端数据库层面做

在数据库端库内设置 session timeout 时间，对于空闲连接超过一定时间是进行主动释放。

#### 4、在服务端操作系统层面做

我们来看一下操作系统端 `keepalive`参数有哪些，以及如何配置，在 redhat、centOS、kylin 等 linux 操作系统为例。

```shell
root@***:~# cat /proc/sys/net/ipv4/tcp_keepalive_time
7200
root@***:~# cat /proc/sys/net/ipv4/tcp_keepalive_probes
9
root@***:~# cat /proc/sys/net/ipv4/tcp_keepalive_intvl
75
```

1. `tcp_keepalive_time`
   keepalive 探测时间间隔，tcp 连接处于最大的 idle 时长，默认 2 小时，太长了，一般都要设短
2. `tcp_keepalive_probes`
   如果探测失败，peer 没有返回 `ACK`
   , 那么再连续探测次数，默认是 9 次
3. `tcp_keepalive_intvl`
   首次探测失败后，连续 probes 的间隔，默认 75s

### 总结

个人理解其实在 RFC 草案中并没有规定 tcp 的 keepalive 一定要在客户端做还是在服务端做，个人理解是都可以的，对于服务端来说有操作系统层面的参数配置，至少是可以保证服务端的资源不会被无限制占用，对于客户端来说尽量从应用层面去配置探活，而不是修改应用端的操作系统配置，因为其它的程序也会默认使用这些系统参数，深度定制调校的系统除外。



