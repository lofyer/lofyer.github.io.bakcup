---
title: ltsp相关
author: lofyer
layout: post
comments: true
permalink: /httpwiki-phys-ethz-chreadmesetting_up_an_ltsp_server_for_diskless_clients/
categories:
  - Linux Admin
---
参考：

https://help.ubuntu.com/community/UbuntuLTSP

安装：  
apt-get install ltsp-server-standalone

启动要素：  
nbd-server  
dhcpd  
tftp-hpa  
openssh-server

绑定客户端地址：  
[dhcpd.conf]  
host client201 {  
hardware ethernet 08:00:27:89:70:01;  
fixed-address 192.168.0.201;  
}  
另外一种是在启动pxe配置文件中指定

http://wiki.phys.ethz.ch/readme/setting\_up\_an\_ltsp\_server\_for\_diskless_clients

session & windows  
[/usr/share/xsession/*]  
Exec=/root/.xsession

[/root/.xsession]  
#!/bin/bash  
gnome-session &  
firefox  
logout

获取session list  
[/usr/share/ldm/ldminfod]  
[/etc/X11/xinit/Xsession]  
[/etc/X11/Xsession] **failsafe  
[/etc/X11/xdm/Xsession]  
[/usr/lib/X11/xdm/Xsession]  
*[/usr/share/xsession]

**default session  
exported by Xsession.d  
echo $DEFAULTS_PATH  
/usr/share/gconf/

修改default session  
[/var/lib/tftp.../lts.conf]  
LDM_SESSION=&#8221;gnome-session &;firefox;logout&#8221;

FatClient  
[/var/lib/tftp.../lts.conf]  
[default]  
LDM_DIRECTX=true

[00:A1:08:EB:43:27]  
LTSP_FATCLIENT=false

AutoLogin  
[/var/lib/tftp.../lts.conf]  
[Default]  
LDM_AUTOLOGIN = True

[192.168.1.101]  
LDM_USERNAME = user1  
LDM_PASSWORD = password1

[192.168.1.102]  
LDM_USERNAME = user2  
LDM_PASSWORD = password2

一些参考配置  
[lts.conf]  
\# Global defaults for all clients  
\# if you refer to the local server, just use the  
\# &#8220;server&#8221; keyword as value  
\# see lts_parameters.txt for valid values  
################  
[default]  
X\_COLOR\_DEPTH=24  
LOCALDEV=True  
SOUND=True  
USE\_LOCAL\_SWAP=True  
NBD_SWAP=False  
SYSLOG_HOST=server  
#XKBLAYOUT=de  
SCREEN_02=shell  
SCREEN_03=shell  
SCREEN_04=shell  
SCREEN_05=shell  
SCREEN_06=shell  
SCREEN_07=ldm  
\# LDM_DIRECTX=True allows greater scalability and performance  
\# Turn this off if you want greater security instead.  
LDM_DIRECTX=True  
\# LDM_SYSLOG=True writes to server&#8217;s syslog  
LDM_SYSLOG=True