---
title: ovirt notes
author: lofyer
layout: post
permalink: /ovirt-notes/
simplecatch-sidebarlayout:
  - 
duoshuo_thread_id:
  - 1234836220387786908
categories:
  - Virtualization
---
Ovirt  
Just make some notes:  
\# need net  
qemu-kvm -m 1024 -localtime -M pc -smp 2 -drive file=ovirt.qcow2,cache=writeback,boot=on -boot d -name kvm-ovirt,process=ovirt -usb -usbdevice tablet  
Ovirt provided a fedora.iso with node in it. Ao is building the engine.

#on virtfan  
spicec -h localhost -p 5910

node应该是自动加入engine的，结果是现在能加入，但是不能安装  
nmap后engine确实是有8443，node只有22，安装过程是交互的（node通过外网获取安装包，返回状态到engine）

改变端口为443，按照troubleshooting更改engine上的nfs服务。f17上nfs默认建立服务为v4，要改为v3，并且添加group和user id分别为36的vdsm与kvm，用提供的脚本测试下。  
<a href="http://wiki.ovirt.org/wiki/Troubleshooting_NFS_Storage_Issues" title="fix the nfs" target="_blank">Troubleshooting_NFS_Storage_Issues</a>

ok，等明天加入节点就可以了。

Building ovirt from source  
<a href="http://wiki.ovirt.org/wiki/Building_oVirt_engine" title="with cmdline" target="_blank">with cmdline</a>  
<a href="http://wiki.ovirt.org/wiki/Building_Ovirt_Engine/IDE" title="within an ide" target="_blank">within an ide</a>  
<a href="https://community.jboss.org/en/tools/blog/2011/02/21/getting-started-with-jboss-tools-jboss-maven-integration-and-weld" title="install jboss maven plugin on eclipse" target="_blank">install jboss maven plugin on eclipse</a>

here&#8217;s the engine arch  
[<img src="http://69.164.197.168/wp-content/uploads/2012/10/Engine-arch.png" alt="" title="Engine-arch" width="955" height="540" class="alignnone size-full wp-image-1606" />][1]

and here&#8217;s the engine-core arch  
[<img src="http://69.164.197.168/wp-content/uploads/2012/10/Engine-arch2.png" alt="" title="Engine-arch2" width="934" height="650" class="alignnone size-full wp-image-1607" />][2]

运行engine-manage-domain需要ipa-server，类似windows的ActiveDirectory，需要不同于engine的主机上安装ipa  
<a href="http://blog.jaurola.fi/2011/12/28/implementing-ipa-server-with-windowslinux-environment" title="ipa-server" target="_blank">安装ipa-server</a>

 [1]: http://69.164.197.168/wp-content/uploads/2012/10/Engine-arch.png
 [2]: http://69.164.197.168/wp-content/uploads/2012/10/Engine-arch2.png