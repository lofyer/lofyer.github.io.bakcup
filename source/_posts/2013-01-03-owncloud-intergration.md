---
title: owncloud webdav intergration
author: lofyer
layout: post
comments: true
duoshuo_thread_id:
  - 1234836220387786890
categories:
  - OwnCloud
---
安装好owncloud后，可以使用webdav协议进行远程挂载，比如

<pre>mount -t davfs {http://localhost/remote.php/webdav,http://localhost/files/webdav.php} /mnt
</pre>

2003上需启动webclient服务  
而在2008上, webclient被集成到desktop experience组件中, 从service manager->feature中添加desktop experience,并且修改注册表HKEY\_LOCAL\_MACHINE->SYSTEM->CurrentControlSet->Services->WebClient->Parameters->BasicAuthLevel 的值为 2，重启服务  
此时可映射网络驱动器，钩选“connect using different credentials”，输入其用户名密码即可  
或者命令行

<pre>NET USE *  http://localhost/remote.php/webdav 123456 /user:admin
</pre>

PS：  
1. owncloud第三方app WebDev安装后会造成不能添加用户的麻烦  
2. 修改owncloud目录权限为apache.apache或者www-data.www-data以使.htaccesss生效