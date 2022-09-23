---
title: "Ceph高可用部署"
date: 2022-09-15T11:30:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["运维","存储","云原生"]
author: "Esqer"
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: true
description: "本文介绍如何高可用部署Ceph集群，Ceph的主要组件"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
editPost:
    URL: "https://github.com/EsqerYasen/EsqerYasen.github.io/content"
    Text: "Edit This Page" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Ceph 学习文档

本教程用官网最近的cephadm来搭建ceph集群。

> 概览
> 1.ceph的组件和功能
> 2.ceph的数据读写流程
> 3.使用ceph-deploy安装一个最少三个节点的ceph集群
>  推荐3个或以上的磁盘作为专用osd 
> 4.测试ceph的rbd使用

## 1·Ceph组件和功能

### 组件

- **Ceph OSDs**: ( Ceph OSD ）object storage daemon的功能是存储数据，处理数据的复制、恢复、回填、再均衡，并通过检查其他OSD 守护进程的心跳来向 Ceph Monitors 提供一些监控信息。当 Ceph 存储集群设定为有2个副本时，至少需要2个 OSD 守护进程，集群才能达到 `active+clean` 状态（ Ceph 默认有3个副本，但你可以调整副本数）。
- **Monitors**: 维护着展示集群状态的各种图表，包括监视器图、 OSD 图、归置组（ PG ）图、和 CRUSH 图。 Ceph 保存着发生在Monitors 、 OSD 和 PG上的每一次状态变更的历史信息（称为 epoch ）。

- **MDSs**: Ceph 元数据服务器为 Ceph 文件系统存储元数据（也就是说，Ceph 块设备和 Ceph 对象存储不使用MDS ）。元数据服务器使得 POSIX 文件系统的用户们，可以在不对 Ceph 存储集群造成负担的前提下，执行诸如 `ls`、`find` 等基本命令。
- **CephMgr**：在一个主机上的守护进程，负责运行指标，运行状态，性能负载，

### 其他术语：

RADOS：多个主机组成的存储集群，即可靠，自动化，分布式的对象存储系统。

File:  就是普通文件，ObjectRADOS看到的对象，Object与File的区别是， Object的最大尺寸由RADOS限定(通常为2MB或4MB) ，以便实现底层存储的组织管理。因此，当上层应用向RADOS存入尺寸很大的File时，需要将File切分成统一大小的一系列Objet (最后一个的大小可以不同)进行存储。

librados：RADOS集群的API，支持大部分主流语言。

Pool：存储池，大小取决于底层的存储空间。

PG：placeholder group，一个pool（存储池）内可以有多个PG，pool个pg都是抽象的逻辑概念，可以通过公示计算。PG的用途是对Object的存储进行组织和位置映射的。具体而言，一个PG负责组织若干个Object，但一个Obiect只能被映射到一个PG中，即PG和Object之间是“一对多”的映射关系。同时，一个PG会被映射到n个OSD上，而每个OSD上都会承载大量的PG，即PG和OSD之间是“多对多”的映射关系。在实践当中，n至少为2，如果用于生产环境，则至少为3。一个OSD上的PG可达到数百个。事实上， PG数量的设置关系到数据分布的均匀性问题。

OSD daemon：默认每2秒发送状态数据给monitor，（同时监控组内其他OSD的状态）（up 可以提供IO，down不能提供，in有数据，out没有数据）

PG和OSD之间的关系通过CRUSH算法得出的。常规这三个 OSD daemon 可以在一台机器上，也可以在不同机器上；那么根据 CRUSH 算法会尽可能的保证一个平衡，就是不在同一个机器上；毕竟Ceph中的数据是一个为平衡的状态，一切都是通过CRUSH 算法来实现的数据平衡，而 PG 本身是个有序列表，位于第一的位置是 master；这个列表的产生是由 monitor 来产生的；

### 寻址流程

**File->Object映射**
这次映射的目的是，将用户要操作的File映射为RADOS能够处理的Object，其十分简单，本质上就是按照Object的最大尺寸（默认4M）对File进行切分，相当于磁盘阵列中的条带化过程。这种切分的好处有两个:一是让大小不限的File变成具有一致的最大尺寸、可以被RADOS高效管理的Object;二是让对单一File实施的串行处理变为对多个Object实施的并行化处理。
每一个切分后产生的Object将获得唯一的oid,即Object ID,其产生方式也是线性映射,极其简单。
**Object →PG映射**
在File被映射为1个或多个Object之后,就需要将每个Object独立地映射到1个PG中去。这个映射过程也很简单,如图所示,其计算公式如下:
Hash(oid) & mask -> pgid
由此可见,其计算由两步组成。首先,使用Ceph系统指定的一个静态哈希算法计算oid的哈希值,将oid映射为一个近似均匀分布的伪随机值。然后,将这个伪随机值和mask按位相与,得到最终的PG序号(pgid) 。根据RADOS的设计,给定PG的总数为m(m应该为2的整数幂),则mask的值为m-1。因此,哈希值计算和按位与操作的整体结果事实上是从所有m个PG中近似均匀地随机选择1个。基于这一机制,当有大量Object和大量PG时, RADOS能够保证Object和PG之间的近似均匀映射。又因为Object是由File切分而来的,大部分Object的尺寸相同,因此,这一映射最终保证了各个PG中存储的Object的总数据量近似均匀。
这里反复强调了“大量” ,意思是只有当Object和PG的数量较多时,这种伪随机关系的近似均匀性才能成立, Ceph的数据存储均匀性才有保证。为保证“大量”的成立,一方面, Object的最大尺寸应该被合理配置,以使得同样数量的File能够被切分成更多的Object;另一方面, Ceph也推荐PG总数应该为OSD总数的数百倍,以保证有足够数量的PG可供映射。
**PG→ OSD映射**
第3次映射就是将作为Object的逻辑组织单元的PG映射到数据的实际存储单元OSD上。RADOS采用一个名为CRUSH的算法,将pgid代入其中,然后得到一组共n个OSD。这n个OSD共同负责存储和维护一个PG中的所有Objecto前面提到过, n的数值可以根据实际应用中对于可靠性的需求而配置,在生产环境下通常为3。具体到每个OSD,则由其上运行的OSD Daemon负责执行映射到本地的Object在本地文件系统中的存储、访问、元数据维护等操作。
和“Object →PG"映射中采用的哈希算法不同, CRUSH算法的结果不是绝对不变的,而会受到其他因素的影响。其影响因素主要有两个。
一是当前系统状态,也就是在前面有所提及的集群运行图。当系统中的OSD状态、数量发生变化时,集群运行图也可能发生变化,而这种变化将会影响到PG与OSD之间的映射关系。
二是存储策略配置。这里的策略主要与安全相关。利用策略配置,系统管理员可以指定承载同一个PG的3个OSD分别位于数据中心的不同服务器或机架上,从而进一步改善存储的可靠性。
因此,只有在系统状态和存储策略都不发生变化的时候, PG和OSD之间的映射关系才是固定不变的。在实际使用中,策略一经配置通常不会改变。而系统状态的改变或是因为设备损坏,或是因为存储集群规模扩大。好在Ceph本身提供了对这种变化的自动化支持,因而,即便PG与OSD之间的映射关系发生了变化,也并不会对应用产生影响。事实上, Ceph正是利用了CRUSH算法的动态特性,可以将一个PG根据需要动态迁移到不同的OSD组合上,从而自动化地实现高可靠性、数据分布再平衡等特性。
之所以在此次映射中使用CRUSH算法,而不使用其他哈希算法,一方面原因是CRUSH算法具有上述可配置特性,可以根据管理员的配置参数决定OSD的物理位置映射策略;另一方面原因是CRUSH算法具有特殊的“稳定性" ,也即,当系统中加入新的OSD,导致系统规模增大时,大部分PG与OSD之间的映射关系不会发生改变,只有少部分PG的映射关系会发生变化并引发数据迁移。这种可配置性和稳定性都不是普通哈希算法所能提供的。因此, CRUSH算法的设计也是Ceph的核心内容之一。
**至此为止, Ceph通过3次映射,完成了从File到Object. Object到PG,PG再到OSD的整个映射过程。从整个过程可以看到,这里没有任何的全局性查表操作需求。至于唯一的全局性数据结构:集群运行图。它的维护和操作都是轻量级的,不会对系统的可扩展性、性能等因素造成影响**。

### 存储过程总结：

1.计算文件到对象的映射

2.通过哈希算法计算计算出文件对应的pool的PG

3.通过CRUSH把对象映射到PG中的OSD

4.PG种的OSD将对象写入到磁盘

5.主OSD将数据同步到备份OSD，待备份OSD返回确认

6.主OSD的到备份OSD写完操作以后给客户的返回写入成功

## 2. ceph的读写流程

当某个客户端需要向Ceph集群写入一个File时，首先需要在本地完成寻址流程，将File变为一个Object，然后找出存储该Object的一组共3个OSD，这3个OSD具有各自不同的序号，序号最靠前的那个OSD就是这一组中的Primary OSD，而后两个则依次Secondary OSD和Tertiary OSD。
找出3个OSD后，客户端将直接和Primary OSD进行通信，发起写入操作(步骤1)。 Primary OSD收到请求后，分别向Secondary OSD和Tertiary OSD发起写人操作(步骤2和步骤3)。当Secondary OSD和Tertiary OSD各自完成写入操作后，将分别向Primary OSD发送确认信息(步骤4和步骤5)。当Primary OSD确认其他两个OSD的写入完成后，则自己也完成数据写入，并向客户端确认Object写入操作完成(步骤6)。
之所以采用这样的写入流程，本质上是为了保证写入过程中的可靠性，尽可能避免出现数据丢失的情况。同时，由于客户端只需要向Primary OSD发送数据，因此在互联网使用场景下的外网带宽和整体访问延迟又得到了一定程度的优化。
当然，这种可靠性机制必然导致较长的延迟，特别是，如果等到所有的OSD都将数据写入磁盘后再向客户端发送确认信号，则整体延迟可能难以忍受。因此， Ceph可以分两次向客户端进行确认。当各个OSD都将数据写入内存缓冲区后，就先向客户端发送一次确认，此时客户端即可以向下执行。待各个OSD都将数据写入磁盘后，会向客户端发送一个最终确认信号，此时客户端可以根据需要删除本地数据。
分析上述流程可以看出，在正常情况下，客户端可以独立完成OSD寻址操作，而不必依赖于其他系统模块。因此，大量的客户端可以同时和大量的OSD进行并行操作。同时，如果一个File被切分成多个Object，这多个Object也可被并行发送至多个OSD上。
从OSD的角度来看，由于同一个OSD在不同的PG中的角色不同，因此，其工作压力也可以被尽可能均匀地分担，从而避免单个OSD变成性能瓶颈。

**问：为什么要设计三层映射而不是一层？**

答：如果将object直接映射到一组OSD上，如果这种算法是固定的哈希算法，则意味着一个object被固定映射在一组OSD上，当其中一个OSD损坏时，object也无法部署到新的OSD上(因为映射函数不允许)。

如果设计一个动态算法(例如CRUSH算法)来完成这一映射，结果将是各个OSD所处理的本地元数据暴增，由此带来的计算复杂度和维护工作量也是难以承受的。

综上所诉，引入PG的好处至少有二：一方面试下呢object和OSD之间的动态映射，从而为Ceph的可靠性、自动化等特性的实现留下了空间；另一方面也有效简化了数据的存储组织，大大降低了系统的维护管理开销。

### 1.准备工作

1. 时间同步`

   安装ntpdate（时间同步工具）

   ```shell
   # apt install ntpate
   ```

   ```shell
   0 * * * * ntpdate time1.aliyun.com
   ```

   ```shell
   echo  '0 * * * * ntpdate time1.aliyun.com' >> /var/spool/cron/crontabs/root
   ```

   或者 可以通过

   ```shell
   ansible all -m shell -a "echo  '0 * * * * ntpdate time1.aliyun.com' >> /var/spool/cron/crontabs/root"
   ```

   

2. 关闭 selinux 和防火墙

   ```shell
   root@node1:~# sudo ufw status  ##查看状态
   Status: inactive
   root@node1:~# sudo ufw disable
   Firewall stopped and disabled on system startup  ##禁用
   root@node1:~#
   ```

   

3. 配置域名解析或通过 DNS 解析

   ```shell
   root@node1:~# cat /etc/hosts
   127.0.0.1	localhost
   root@node1:~# hostnamectl set-hostname 对应的名称
   ## 以下是新增的 可以按照自己的习惯配置
   192.168.106.101  node1
   192.168.106.102  node2
   192.168.106.103  node3
   ```

   4. 安装python

      ```shell
      root@node1:~# apt install python  ##python2
      ```

   5. 源修改成国内源  -- 具体步骤自行百度

      ```yml
      https://mirrors.aliyun.com/ceph/ #阿里云镜像仓库
      http://mirrors.163.com/ceph/ #网易镜像仓库 
      https://mirrors.tuna.tsinghua.edu.cn/ceph/ #清华大学镜像源
      ```

      ceph用到的端口 (防火墙和安全中记得放开)

      ```yaml
      Ceph Monitor：启用 Ceph MON 服务或端口 6789 (TCP)。
      Ceph OSD 或元数据服务器：启用 Ceph OSD/MDS 服务或端口 6800-7300 (TCP)。
      iSCSI 网关：打开端口 3260 (TCP)。
      对象网关：打开对象网关通讯所用的端口。此端口在 /etc/ceph.conf 内以 rgw frontends = 开头的行中设置。HTTP 的默认端口为 80，HTTPS (TCP) 的默认端口为 443。
      NFS Ganesha：默认情况下，NFS Ganesha 使用端口 2049（NFS 服务、TCP）和 875 （rquota 支持、TCP）。
      SSH：打开端口 22 (TCP)。
      NTP：打开端口 123 (UDP)。
      ```

      

   ### 2.搭建ceph集群

   #### 安装cephadm

   ```shell
   root@node1:~#  wget https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm ## node1 管理节点上执行
   root@node1:~#  chmod +x cephadm
   root@node1:~# ./cephadm add-repo --release pacific  ##设置要安装的版本
   root@node1:~#  which cephadm   ##确认是否安装成功
   ```

   2. #### 初始化集群

      ```shell
      root@node1:~# cephadm bootstrap --mon-ip 192.168.106.101   ##ceph集群第一个节点的ip
      ```

      初始化完了以后就可以访问dashboard了 地址 ：https://node1:8443/#/dashboard 访问用户密码上一步生成

   3. #### 添加其他节点和其他组件

      ```shell
      root@node1:~# ssh-keygen
      ## 配置免密通信
      root@node1:~#  ssh-copy-id -f -i /etc/ceph/ceph.pub root@node2
      root@node1:~#  ssh-copy-id -f -i /etc/ceph/ceph.pub root@node3
      ## 添加node
      root@node1:~#  ceph orch host add node2 192.168.106.102
      root@node1:~#  ceph orch host add node3 192.168.106.103
      ## 添加osd
      root@node1:~#  ceph orch daemon add osd node1:/dev/sdb
      root@node1:~#  ceph orch daemon add osd node1:/dev/sdb
      root@node1:~#  ceph orch daemon add osd node3:/dev/sdb
      ```

   4. #### 测试

      ```shell
      root@node1:~#  ceph fs volume create testfs  ##添加测试fs
      root@node1:~#  ceph orch apply mds testfs --placement="3" ##设置备份数
      root@node1:~#   ceph orch daemon add mds testfs node1
      root@node1:~#   ceph mds stat
      
      ## 在集群之外的或者任意机器上操作
      root@node4:~#  apt install ceph-common -y
      node1初始化集群的节点操作
      root@node1:~#  scp /etc/ceph/ceph.client.admin.keyring user@node4:/etc/ceph
      
      ##  集群之外的clinet或者测试节点执行
      root@node4:~#  mount -t ceph node1:/ /mnt/testfs -o name=admin,secret=AQAoJjBh7OPVNhAAQZyzLhDfgSj+KPmeU5RVlA==,fs=testfs  
      root@node4:~#  mount -t ceph node2:/ /mnt/cephfs -o name=admin,secret=AQAoJjBh7OPVNhAAQZyzLhDfgSj+KPmeU5RVlA==,fs=testfs
      root@node4:~#  df -h
      Filesystem                  Size  Used Avail Use% Mounted on
      udev                        1.4G     0  1.4G   0% /dev
      tmpfs                       293M  1.2M  292M   1% /run
      ....
      192.168.106.101:/            18G 1000M   17G   6% /mnt/testfs  
      192.168.106.102:/            18G 1000M   17G   6% /mnt/cephfs
      
      root@node4:~#  cd /mnt/cephfs
      root@node4:/mnt/cephfs#  dd if=/dev/zero of=test bs=1M count=100 ##生成文件
      这时候文件是直接写在ceph集群上看了， 可以通过dashboard观察👀。
      ```

      ![](https://github.com/EsqerYasen/ImgBox/blob/main/ceph-dashboard.png)

