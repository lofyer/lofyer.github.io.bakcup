---
title: 7. Gluster简述
author: lofyer
layout: post
comments: true
videourl:
  - 
categories:
  - 云端日记
---
这一节会简述下glusterfs功能和特性。

glusterfs设计符合奥卡姆剃刀原则，即“若无必要，勿增实体”。

<a href="http://blog.lofyer.org/5-2-cloud-gluster-src/cloud-5-2-1/" rel="attachment wp-att-2999"><img class="size-full wp-image-2999 aligncenter" alt="cloud-5.2-1" src="http://io.lofyer.org/uploads/cloud-5.2-1.png" width="787" height="486" /></a>

## 功能特性

gluster集群组成的存储可以通过NFS、CIFS、Glusterfs、HTTP、FTP协议交给用户使用；

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/glusterfs_architecture/" rel="attachment wp-att-3007"><img src="http://io.lofyer.org/uploads/GlusterFS_Architecture.png" alt="GlusterFS_Architecture" width="800" height="576" class="alignnone size-full wp-image-3007" /></a>

gluster支持四种存储逻辑组合：Striped、Replicated、Striped-Replicated、Distributed；

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/distributed_volume/" rel="attachment wp-att-3013"><img src="http://io.lofyer.org/uploads/Distributed_Volume.png" alt="Distributed_Volume" width="660" height="360" class="alignnone size-full wp-image-3013" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/distributed_striped_volume/" rel="attachment wp-att-3012"><img src="http://io.lofyer.org/uploads/Distributed_Striped_Volume.png" alt="Distributed_Striped_Volume" width="660" height="400" class="alignnone size-full wp-image-3012" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/distributed_replicated_volume/" rel="attachment wp-att-3010"><img src="http://io.lofyer.org/uploads/Distributed_Replicated_Volume.png" alt="Distributed_Replicated_Volume" width="660" height="400" class="alignnone size-full wp-image-3010" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/distributed_striped_replicated_volume/" rel="attachment wp-att-3011"><img src="http://io.lofyer.org/uploads/Distributed_Striped_Replicated_Volume.png" alt="Distributed_Striped_Replicated_Volume" width="660" height="460" class="alignnone size-full wp-image-3011" /></a>

支持跨网备份（geo-replication）；

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/geo-rep_lan/" rel="attachment wp-att-3017"><img src="http://io.lofyer.org/uploads/Geo-Rep_LAN.png" alt="Geo-Rep_LAN" width="676" height="241" class="alignnone size-full wp-image-3017" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/geo-rep_wan/" rel="attachment wp-att-3015"><img src="http://io.lofyer.org/uploads/Geo-Rep_WAN.png" alt="Geo-Rep_WAN" width="628" height="206" class="alignnone size-full wp-image-3015" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/geo-rep03_internet/" rel="attachment wp-att-3018"><img src="http://io.lofyer.org/uploads/Geo-Rep03_Internet.png" alt="Geo-Rep03_Internet" width="659" height="241" class="alignnone size-full wp-image-3018" /></a>

<a href="http://blog.lofyer.org/5-2-cloud-gluster-des/geo-rep04_cascading/" rel="attachment wp-att-3019"><img src="http://io.lofyer.org/uploads/Geo-Rep04_Cascading.png" alt="Geo-Rep04_Cascading" width="709" height="516" class="alignnone size-full wp-image-3019" /></a>

统一对象视图，与UNIX设计哲学类似，所有皆对象；

跨平台兼容性高，可作为hadoop、openstack、ovirt、Amazon EC的后端；

## 名词解释

砖块（brick）：即服务器节点上导出的一个目录，作为glusterfs的最基本单元；

扩展属性（xattr）：用户/程序以之将数据和元数据关联起来；

GFID：glusterfs中的每一个文件或者目录都有一个独立的128位GFID，与普通文件系统中的inode类似；

glusterd：运行于所有服务器中的管理守护程序；