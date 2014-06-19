---
title: 9. glusterfs应用示例及技巧
author: lofyer
layout: post
comments: true
permalink: /5-4-gluster-trick/
videourl:
  - 
categories:
  - 云端日记
---
## 文件权限

glusterfs在创建卷时会更改砖块所有者为root.root，对于某些应用请注意更改砖块目录所有者（比如在*/etc/rc.local*中添加chown，不要更改砖块下隐藏目录.glusterfs）。

## 砖块分割

前文所述的砖块划分方式在某些场景下不是很好，可以在/brickX下创建目录，比如data1，同时在创建glusterfs卷的时候使用HOST:/brickX/data1作为砖块，以合理利用存储空间。

## normal、replica、striped卷组合

砖块的划分排序：striped（normal）优先，replica在striped（normal）基础上做冗余；计算大小时，同一replica组中的brick进行合并（多个算作一个），一个striped组可看做一个有效块，。  
假设我们有4个主机，8个砖块，每个砖块都是5GB，如下图，

<a href="http://blog.lofyer.org/5-4-gluster-trick/brick-base/" rel="attachment wp-att-3294"><img src="http://blog.lofyer.org/wp-content/uploads/brick-base.png" alt="brick-base" width="702" height="268" class="alignnone size-full wp-image-3294" /></a>

创建卷时，使用如下命令：

<pre># gluster volume create gluster-vol1 stripe 2 replica 2 host1:/brick1 host1:/brick2 host2:/brick1 host2:/brick2 host3:/brick1 host3:/brick2 host4:/brick1 host4:/brick2 force</pre>

则会进行下列组合：

<a href="http://blog.lofyer.org/5-4-gluster-trick/brick-1/" rel="attachment wp-att-3301"><img src="http://blog.lofyer.org/wp-content/uploads/brick-1.png" alt="brick-1" width="712" height="474" class="alignnone size-full wp-image-3301" /></a>

创建卷时，使用如下命令：

<pre># gluster volume create gluster-vol1 stripe 2 replica 2 host1:/brick1 host2:/brick1 host3:/brick1 host4:/brick1 host1:/brick2 host2:/brick2 host3:/brick2 host4:/brick2 force</pre>

则会进行下列组合（注意颜色，可改为实虚线）：

<a href="http://blog.lofyer.org/5-4-gluster-trick/brick-2/" rel="attachment wp-att-3304"><img src="http://blog.lofyer.org/wp-content/uploads/brick-2.png" alt="brick-2" width="712" height="474" class="alignnone size-full wp-image-3304" /></a>

## 作为nfs挂载

**由于glusterfs占用了2049端口，所以其与nfs server一般不能共存于同一台服务器，除非更改nfs服务端口。**

<pre>mount -t nfs -o vers=3 server1:/volume1 /mnt</pre>

## 作为cifs挂载

先在某一服务器或者客户端将起挂载，再以cifs方式导出

<pre title="Content added
 to /etc/smb.conf">[glustertest]
comment = For testing a Gluster volume exported through CIFS
path = /mnt/glusterfs
read only = no
guest ok = yes
</pre>

## 进阶手册

### 1. 裂脑(split-brain)修复

裂脑发生以后，各节点信息可能会出现不一致。可以通过以下步骤查看并修复。

**定位裂脑文件**  
通过命令

<pre># gluster volume heal info split-brain</pre>

或者查看在客户端仍然是Input/Output错误的文件。  
**关闭已经打开的文件或者虚机**  
**确定正确副本**  
**恢复扩展属性**

### 2. 砖块重用

当一个volume正在使用时，你删除了其中一个brick，会出现“/bricks/app or a prefix of it is already part of a volume”，对于3.3版本以后的glusterfs有此问题。  
解决方法：

<pre># setfattr -x trusted.glusterfs.volume-id $brick_path
# setfattr -x trusted.gfid $brick_path
# rm -rf $brick_path/.glusterfs</pre>

### 3. 扩展

对于中等以上规模的部署，需要使用dns服务器去解析各个节点以免去修改hosts文件的麻烦。