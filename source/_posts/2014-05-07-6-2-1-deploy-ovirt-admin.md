---
title: 11. 搭建oVirt虚拟化平台
author: lofyer
layout: post
comments: true
videourl:
  - 
categories:
  - 云端日记
---
对于初次使用oVirt的用户，建议使用此种搭建方式，**太折腾的话就吓走好多目标读者了**。

使用之前的四台机器，分别为gs1.lofyer.org，gs2.lofyer.org，gs3.lofyer.org和gs4.lofyer.org，其中，将gs1作为管理机安装ovirt-engine，其余三台作为节点（node），存储使用已经创建好的glusterfs。

## 准备

**存储可以使用之前的glusterfs，方式为NFS_V3，注意将brick的权限设置为vdsm.kvm或者36:36。**

<pre class="lang:default decode:true " ># gluster volume create gluster-vol1 replica 2 gs1.example.com:/gluster_brick0 gs2.example.com:/gluster_brick0 gs3.example.com:/gluster_brick0 gs4.example.com:/gluster_brick0 gs1.example.com:/gluster_brick1 gs2.example.com:/gluster_brick1 gs3.example.com:/gluster_brick1 gs4.example.com:/gluster_brick1 force</pre>

<a href="http://blog.lofyer.org/11-cloud-ha-ovirt/gluster-ovirt/" rel="attachment wp-att-3307"><img src="http://lofyer.github.io/uploads/gluster-ovirt.png" alt="gluster-ovirt" width="712" height="537" class="alignnone size-full wp-image-3307" /></a>

**由于engine以及node的网络服务依赖于network而非NetworkManager，我们需要启用前者禁用后者，在每一台服务器上都进行如下类似配置修改网络。**

<pre title="/etc/sysconfig/network-scripts/ifcfg-eth0">NAME=eth0
DEVICE=eth0
ONBOOT=yes
BOOTPROTO=static
# 注意修改此处的IP
IPADDR=192.168.10.101
NETMASK=255.255.255.0
GATEWAY=192.168.10.1
DNS1=192.168.10.1</pre>

<pre># chkconfig NetworkManager off
# chkconfig network on
# service NetworkManager stop; service network restart</pre>

## 添加repo

<pre># yum localinstall http://resources.ovirt.org/releases/ovirt-release.noarch.rpm
# yum install ovirt-hosted-engine-setup</pre>

或者手动添加：

<pre title="/etc/yum.repos.d/ovirt.repo">[ovirt-stable]
name=Latest oVirt Releases
baseurl=http://ovirt.org/releases/stable/rpm/EL/$releasever/
enabled=1
skip_if_unavailable=1
gpgcheck=0

[ovirt-3.4-stable]
name=Latest oVirt 3.4.z Releases
baseurl=http://ovirt.org/releases/3.4/rpm/EL/$releasever/
enabled=1
skip_if_unavailable=1
gpgcheck=0

[epel]
name=Extra Packages for Enterprise Linux 6 - $basearch
#baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&#038;arch=$basearch
failovermethod=priority
enabled=1
includepkgs=epel-release,python-uinput,puppet,python-lockfile,python-cpopen,python-ordereddict,python-pthreading,python-inotify,python-argparse,novnc,python-ply,python-kitchen,python-daemon,python-websockify,livecd-tools,spice-html5,mom
gpgcheck=0

[ovirt-glusterfs-epel]
name=GlusterFS is a clustered file-system capable of scaling to several petabytes.
baseurl=http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/epel-$releasever/$basearch/
enabled=1
skip_if_unavailable=1
gpgcheck=0

[ovirt-glusterfs-noarch-epel]
name=GlusterFS is a clustered file-system capable of scaling to several petabytes.
baseurl=http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/epel-$releasever/noarch
enabled=1
skip_if_unavailable=1
gpgcheck=0
</pre>

## 安装管理节点。

在gs1上运行如下命令。

<pre title="setup webadmin"># yum install ovirt-engine
# engine-setup --offline
[ INFO  ] Stage: Initializing
[ INFO  ] Stage: Environment setup
          Configuration files: ['/etc/ovirt-engine-setup.conf.d/10-packaging.conf']
          Log file: /var/log/ovirt-engine/setup/ovirt-engine-setup-20140508054649.log
          Version: otopi-1.2.0 (otopi-1.2.0-1.el6)
[ INFO  ] Stage: Environment packages setup
[ INFO  ] Stage: Programs detection
[ INFO  ] Stage: Environment setup
[ INFO  ] Stage: Environment customization
         
          --== PRODUCT OPTIONS ==--
         
         
          --== PACKAGES ==--
         
         
          --== NETWORK CONFIGURATION ==--
         
          Host fully qualified DNS name of this server [gs1.lofyer.org]: 
          Setup can automatically configure the firewall on this system.
          Note: automatic configuration of the firewall may overwrite current settings.
          Do you want Setup to configure the firewall? (Yes, No) [Yes]: 
          The following firewall managers were detected on this system: iptables
          Firewall manager to configure (iptables): iptables
[ INFO  ] iptables will be configured as firewall manager.
         
          --== DATABASE CONFIGURATION ==--
         
          Where is the Engine database located? (Local, Remote) [Local]: 
          Setup can configure the local postgresql server automatically for the engine to run. This may conflict with existing applications.
          Would you like Setup to automatically configure postgresql and create Engine database, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
         
          --== OVIRT ENGINE CONFIGURATION ==--
         
          Application mode (Both, Virt, Gluster) [Both]: 
          Default storage type: (NFS, FC, ISCSI, POSIXFS) [NFS]: 
          Engine admin password: 
          Confirm engine admin password: 
         
          --== PKI CONFIGURATION ==--
         
          Organization name for certificate [lofyer.org]: 
         
          --== APACHE CONFIGURATION ==--
         
          Setup can configure apache to use SSL using a certificate issued from the internal CA.
          Do you wish Setup to configure that, or prefer to perform that manually? (Automatic, Manual) [Automatic]: 
          Setup can configure the default page of the web server to present the application home page. This may conflict with existing applications.
          Do you wish to set the application as the default page of the web server? (Yes, No) [Yes]: 
         
          --== SYSTEM CONFIGURATION ==--
         
          Configure WebSocket Proxy on this machine? (Yes, No) [Yes]: 
          Configure an NFS share on this server to be used as an ISO Domain? (Yes, No) [Yes]: no
         
          --== MISC CONFIGURATION ==--
         
         
          --== END OF CONFIGURATION ==--
         
[ INFO  ] Stage: Setup validation
         
          --== CONFIGURATION PREVIEW ==--
         
          Engine database name                    : engine
          Engine database secured connection      : False
          Engine database host                    : localhost
          Engine database user name               : engine
          Engine database host name validation    : False
          Engine database port                    : 5432
          PKI organization                        : lofyer.org
          Application mode                        : both
          Firewall manager                        : iptables
          Update Firewall                         : True
          Configure WebSocket Proxy               : True
          Host FQDN                               : gs1.lofyer.org
          Datacenter storage type                 : nfs
          Configure local Engine database         : True
          Set application as default page         : True
          Configure Apache SSL                    : True
         
          Please confirm installation settings (OK, Cancel) [OK]: ok
[ INFO  ] Stage: Transaction setup
[ INFO  ] Stopping engine service
[ INFO  ] Stopping websocket-proxy service
[ INFO  ] Stage: Misc configuration
[ INFO  ] Stage: Package installation
[ INFO  ] Stage: Misc configuration
[ INFO  ] Initializing PostgreSQL
[ INFO  ] Creating PostgreSQL 'engine' database
[ INFO  ] Configuring PostgreSQL
[ INFO  ] Creating Engine database schema
[ INFO  ] Creating CA
[ INFO  ] Configuring WebSocket Proxy
[ INFO  ] Generating post install configuration file '/etc/ovirt-engine-setup.conf.d/20-setup-ovirt-post.conf'
[ INFO  ] Stage: Transaction commit
[ INFO  ] Stage: Closing up
         
          --== SUMMARY ==--
         
          SSH fingerprint: 1B:FD:08:A2:FD:83:20:8A:65:F5:0D:F6:CB:BF:46:C7
          Internal CA 28:7E:D6:6B:F7:F2:6C:B5:60:27:44:C3:7F:3C:22:63:E5:68:DD:F4
          Web access is enabled at:

http://gs1.lofyer.org:80/ovirt-engine


https://gs1.lofyer.org:443/ovirt-engine

          Please use the user "admin" and password specified in order to login into oVirt Engine
         
          --== END OF SUMMARY ==--
         
[ INFO  ] Starting engine service
[ INFO  ] Restarting httpd
[ INFO  ] Generating answer file '/var/lib/ovirt-engine/setup/answers/20140508054842-setup.conf'
[ INFO  ] Stage: Clean up
          Log file is located at /var/log/ovirt-engine/setup/ovirt-engine-setup-20140508054649.log
[ INFO  ] Stage: Pre-termination
[ INFO  ] Stage: Termination
[ INFO  ] Execution of setup completed successfully</pre>

至此，管理节点安装结束。

接下来，加入节点以及存储域，请参考第13节。