---
layout: post

title: "极客APP-每日一课笔记-ZAB算法与Paxos算法的本质区别是什么"

date: 2020-03-15 13:52:40 +0300

description:  

cover: 'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2020-03-15-%E6%88%AA%E5%B1%8F2020-03-15%E4%B8%8B%E5%8D%883.24.46.png'

color: rgb(65,33,52)

tags: [NoTAG]
---

**主讲 金嘉怡 某知名互联网公司高级研发工程师 **

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2020-03-15-%E6%88%AA%E5%B1%8F2020-03-15%E4%B8%8B%E5%8D%883.24.46.png)

## ZAB 与 Paxos

1990年兰伯特(Leslie Lamport)提出了著名的一致性算法Paxos， 是一种基于消息传递，并且具有高度容错特性的算法。2011 年，ZooKeeper 也提出并实现了一种新的一致性算法：ZAB。随后，这个算法被广泛应用于很多知名的大数据框架中，比如 Hadoop、HBase、Storm、Spark、Flink，等等。那么，ZAB 和传统的 Paxos 相比，有哪些不同呢，它的优势又体现在哪里呢？

## 👿😈关于角色

两种算法都涉及一个“角色”的概念，也就是对于每个物理节点赋予不同的角色

领导者（Leader）

跟随者（Follower）

观察者（Observer）

在zk集群中，ZAB通过选举的方式，来选定某一个节点作为Leader节点，这个节点可以为zk的客户端提供读写服务，其它节点作为Follower或Observer节点，只提供读服务，两者的区别在于Observer节点不参与Leader的选举过程和过半写成功策略，所以增加Observer节点可以在不影响写性能的同时提升集群的读性能，主要用于zk跨机房集群部署，当然，针对此次讨论的问题，我们可以只关注Leader和Follower这两个角色。

## ✅❌关于事务操作

这里指能改变节点状态的操作，以zk为例，一般包括ZNode节点的创建、删除、内容更新等操作，这里的ZNode是什么么，在Zookeeper中用来存储数据的基本单元我们称之为ZNode，每一个Zookeeper节点都会维护它本身需要存储的数据，创建和更新最后一次的时间戳以及子节点数量等信息。

## 🌰🌰举个栗子

例如我们现在需要在一个分布式的共享存储中创建多个目录：/a、/a/b、/b 三个目录。

每次创建目录的操作，我们都认为是一次事务操作。

我们用<[事务ID],"[事务具体的操作]">的形式表示操作

### paxos

1. 在Paxos算法中，Leader节点Node1发起两个操作：<t1,"create /a"> 和 <t2,"create /a/b">
2. 当事务尚未成功时，Node1宕机，使Node2成为新的Leader节点
3. Node2节点发起<t1,"create /b">事务
4. 悲剧重演，此时Node2也宕机了，Node3成为新的Leader节点
5. Node3综合了Node1和Node2的事务：<t1,"create /b"> 和 <t2,"create /a/b">

<u>这时会出现问题，因为/a并不存在，所以/a/b也就无法被创建了。那么ZAB会如何应对这样的场景呢</u>

### ZAB

ZAB保证在任意时刻，集群中都只会有一个节点是Leader节点，所有的事务都由Leader发起，并同步给其它的Follower节点。并且Leader节点会给每一个事务操作一个全局的唯一的ID，称之为ZXID，随后在更新Follower节点时，用的是二阶段提交协议，也就是说只要集群中有超过一半的节点prepare成功就通知他们commit事务。各Follower节点要按当初Leader节点让它们Prepare的顺序来commit事务。这样，前面的情形在ZAB的算法设计中就变成了：

1. Node1创建<0x00010001,"create /a"> 和 <0x00010002,"create /a/b">两个事务
2. 当事务尚未成功时，Node1宕机，使Node2成为新的Leader节点
3. Node2创建<0x00020001,"create /b">事务
4. 悲剧重演，此时Node2也宕机了，Node3成为新的Leader节点
5. Node3汇总到的结果就是<0x00010001,"create /a">、<0x00010002,"create /a/b">、<0x00020001,"create /b"> 三个事务
6. Node3节点依次commit这三个事务请求，就可以成功创建这3个目录了

### 核心的区别

ZAB算法引入了ZXID，解决了事务操作如何保证全局有序的难题，这也奠定了它在大数据分布式领域的霸主地位，但软件设计中并不存在银弹，zk也并不可能适用于所有分布式场景，比如要求写性能可扩展的微服务以及希望能更简单高效管理元数据的Kafka等，也都在寻求更优的设计思路。

## 🔡🔠延伸

### More

一致性算法还有很多，例如

- ElasticSearch中使用的Hash路由算法
- 负载均衡HaProxy中的一致性Hash算法
- 列式存储引擎Cassandra中的Gossip闲话算法
- 键值存储引擎Redis中的Raft选举算法

### conclusion

这些算法有什么区别以及如何选型呢

| 算法            | 关键字                   | 选型举例分析                                                 |
| :-------------- | :----------------------- | :----------------------------------------------------------- |
| Paxos选举算法   | 过半选举、数据副本一致性 | 是最先解决拜占庭将军问题的算法，利用过半选举的机制保证了集群副本的一致性。不过针对微服务中服务注册于发现的场景，对集群的写能力和可用性有较高的要求，Paxos就已经不再适用了 |
| Raft选举算法    | 相对Paxos协议上的简化    | Redis适用Raft实现了自己的分布式一致性，Raft本身和Paxos并没有场景上的区别，更多的是协议上的简化，使得其实现起来工程量会小很多 |
| ZAB原子广播协议 | 适用于离线的海量数据处理 | 例如Hadoop利用Zookeeper保证数据副本的一致性                  |
| Hash路由        | 请求路由                 | ElasticSearch集群接收到为文档创建索引的请求时，需要选择在哪一个Shard上对文档进行索引，这里的Shard是指某一个完整且独立的Lucene索引实例。ElasticSearch采用的是djb2哈希算法，俗称times33，可以简单表示为hash(key)%n，也就是对要索引的文档中默认或指定的key进行哈希操作，然后再对ES集群中Shard的数量n进行取模进而完成请求路由的操作 |
| 一致性Hash算法  | 负载均衡                 | HaProxy使用一致性Hash算法，是用于对服务器连接进行负载均衡的算法。最新的进展是Google近几年发表的一篇有界负载的一致性Hash算法论文 [Consistent Hashing with Bounded Loads](https://arxiv.org/abs/1608.01350)，这篇论文设计的新算法保证了负载均衡一致性和稳定性的同时，在均衡性方面也做出了实质性地改进，之后在HaProxy项目中得以应用，并被证实能显著地节省缓存带宽 |
| Gossip闲话算法  | 最终一致性               | Gossip主要被Cassandra应用于实现它的分布式一致性特性，因为Cassandra框架更看重去中心化和容错的特性，在不违背CAP定理的前提下，只能接受了最终一致性 |


