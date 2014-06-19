---
title: Leapmotion 相关
author: lofyer
layout: post
comments: true
categories:
  - Leapmotion
---
## 在Windows中

夹持器与四轴飞行器都是Arduino-Based，通信就用串口。  
代码放在<a href="http://git.lofyer.org/cgit.cgi/fun/robohand/" title="http://git.lofyer.org/cgit.cgi/fun/robohand/" target="_blank">git.lofyer.org</a>  
感觉VS好难。。试试在gentoo下驱动leapmotion

## 在Gentoo下使用，这个方式应该适用于所有非debian系

先下载sdk包，下载后解压。  
看到里面有个deb的包，用alien转化为tar包

<pre>alien -t Leap-0.8.0-x64.deb
tar xf leap-0.8.0.tgz
cp -irf usr/* /usr/local/
cp -irf lib/udev/rules.d/25-com-leapmotion-leap.rules /etc/udev/rules.d</pre>

然后将普通用户加入plugdev组 
<pre>sudo usermod -a -G plugdev $USER</pre>

刷新组关系，注销当前用户也可

<pre>exec su -l $USER</pre>

运行leapd与LeapControlPanel即可

在64位机器上缺少32位库文件  
freetype.so.6, libasound.so.2

<pre>sudo emerge -avt app-emulation/emul-linux-x86-xlibs
sudo emerge emul-linux-x86-soundlibs</pre>

实例：编译MotionVisualizer

<pre>Make -C Builds/Linux</pre>