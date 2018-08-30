---
layout: post

title: "记一次GlusterFS集群的trouble shooting"

date: 2018-03-15 21:22:20 +0300

description:  

img:  gluster.jpg # Add image post (optional)

tags: [glusterfs]
---

#### 问题描述

今天同事小S在做高可用交接工作时，为了演示高可用的功能，主动停止了测试环境中GLusterFS集群的卷，结果发现GlusterFS的卷无法启动<!-- more -->，报的错误指向10.131.9.116（测试环境中GlusterFS集群的其中一个节点），同事小S查看116的/opt/glusterfs文件系统时发现GlusterFS的数据目录已经损坏。经排查分析，该磁盘可能有损，由于GlusterFS的高可用机制使应用无法感知，所以负责应用开发的同事小Z一直可正常访问文件系统，今天做高可用演示的时候因为主动停止了卷，该问题得以暴露。此时，因为集群volume宕掉，负责应用开发的同事小Z也发现了应用访问文件系统出现了问题

![gluster-trouble-shooting-1]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-1.jpg)

> 测试环境的集群主机均为虚拟机，模拟生产环境的4节点分布式副本集群，116是其中的一个节点



#### 问题解决记录

##### 问题排查与定位

查看116上的glusterfs服务，发现正常

![gluster-trouble-shooting-2]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-2.jpg)

使用gluster peer 命令探测各节点，发现各节点通信情况正常，说明gluster服务本身没有问题（验证glusterd守护进程也正常），排除是软件问题

![gluster-trouble-shooting-3]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-3.jpg)

> 拟定可行的解决思路：
>
> 1. 使用remove-brick的方式将损坏的brick移除，再做出一块GlusterFS的专用盘 用add-brick的方式添加进卷组
>
> 2. 做出一块glusterfs盘，使用replace-brick的方式，不做数据迁移（数据已经损坏）直接替换坏掉的brick

  其中对于思路1，因为我们使用的2副本（replica==2），那么remove或add的brick数量必须为2的整数倍，所以选择思路2进行，付出的代价最小

##### 制作glusterfs的专用盘

> 如果在生产环境，需要进行更换备件，将坏盘替换后往往需要重新做raid以及其它事项，所以对于生产环境的硬盘需要注意更多的底层细节，在本案例中仅以虚拟机为例进行说明

虚拟机中加一块磁盘50G

![gluster-trouble-shooting-4]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-4.jpg)

在系统中查看device路径为/dev/sdc, 确定了我们已添加成功

![gluster-trouble-shooting-5]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-5.jpg)

创建glusterfs的数据目录 `mkdir -p /opt/glusterfs2/data`  损坏的为`/opt/glusterfs/data`，暂时不要删除，以免替换的时候找不到文件（因为今天对坏盘暂未删除VG，所以我还不清楚是否能删除），此步骤没截图

创建新的GlusterFS盘：仍在原卷组（glusterfs_vg）下创建新的逻辑卷`lv_opt_glusterfs2`, (坏掉的为`lv_opt_glusterfs`,对于该类信息的查看可以使用 `vgdisplay -v` 命令查看，这里没截图)

创建的工具使用ssm

![gluster-trouble-shooting-6]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-6.jpg)

该命令直接将创建好的盘进行了挂载`/opt/glusterfs2`,先umount掉，改一下isize的值为512，与其它三个节点（114、115、117）保持一致

![gluster-trouble-shooting-7]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-7.jpg)

挂载命令写入fstab

![gluster-trouble-shooting-8]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-8.jpg)

执行`mount -a` 自动挂载，至此，一块新的glusterfs盘就做好了

##### 使用新的盘替换坏掉的brick

首先，使用强制模式启动volume，因为116的盘坏掉了，是不符合volume的启动检查条件的（也就是问题描述中合平无法直接启动volume的现象），但使用volume命令必须在启动的状态下执行，所以首先使用force选项，将有问题的集群卷组强制启动起来

![gluster-trouble-shooting-9]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-9.jpg)

启动后可以查看卷的信息，坏的brick为`10.131.9.116:opt/gluster`

> 使用下图中命令，用我们做好的盘把坏盘从集群中替掉（这里加的参数commit force 表示不做数据迁移直接替换，一般正常情况可用start参数先做数据迁移，具体可以使用man gluster查看手册

![gluster-trouble-shooting-10]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-10.jpg)

![gluster-trouble-shooting-11]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-11.jpg)

查看替换后的状态，表明已替换成功

![gluster-trouble-shooting-12]({{site.baseurl}}/assets/img/glusterfs-trouble-shooting/gluster-trouble-shooting-12.jpg)

**后续是否要做数据同步或数据再平衡，可根据实际情况决定，操作的命令同样可以通过 man gluster 查阅**
