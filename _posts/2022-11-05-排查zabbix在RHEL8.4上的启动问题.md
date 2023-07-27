---
layout: post

title: "排查zabbix在RHEL8.4上的启动问题"

date: 2022-11-05 17:30:48 +0300

description:  

cover:'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%882.57.44.png'

color: rgb(222,315,333)

tags: [linux]
---

 ![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%882.57.44.png)

### 问题起因

zabbix agent 的大版本是 4，yum 方式安装，操作系统的版本是 RHEL8.4，环境有多台主机，每台的安装过程一样，只有 1 台主机的 zabbix agent 有问题。查看 /var/log/zabbix 下的日志，提示`Zabbix agent cannot initialize module "zbxpcp.so"`，然后不断重新启动，陷入循环。

### 问题排查

bing 上搜集了几个链接，最后发现一个环境相近的案例，直接`yum remove pcp-export-zabbix-agent`果然解决了问题，对比了一下其它几台主机，虽然也有这个包，但其它主机没有遇到这个问题，为了环境保持一致，最后检查了 zabbix agent 正常启动后，又把这个包安装回去了，未发现其它异常。