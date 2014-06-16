---
title: mingw编译remoteviewer
author: lofyer
layout: post
permalink: /mingw%e7%bc%96%e8%af%91remoteviewer/
simplecatch-sidebarlayout:
  - 
duoshuo_thread_id:
  - 1234836220387786776
categories:
  - Virtualization
---
在fedora环境中，需要的有spice-gtk，libusbx，usbredir，remote-review，全部下载最新git源  
spice-gtk，usbredir

http://cgit.freedesktop.org/spice

libusbx  
git://github.com/libusbx/libusbx.git  
remote-viewer  
git.fedorahosted.org/git/virt-viewer.git

libusbx和usbredir：

<pre>mingw32-configure;mingw32-make;mingw32-make install
</pre>

spice-gtk：  
mingw32-configure &#8211;with-sasl=no &#8211;with-audio=gstreamer &#8211;enable-smartcard=no &#8211;with-gtk=2.0 &#8211;without-python  
因为virtviewer要gtk2.0的，所以这也就2.0的；mingw32下没找到pygtk，图省事儿就不要python了，其他的缺啥下啥

<pre>mingw32-make
mingw32-make install
</pre>

virt-viewer：

<pre>mingw32-configure
mingw32-make
mingw32-make install
</pre>

生成remote viewer.exe

安装包  
运行nsiswrapper生成windows安装包

<pre>nsiswrapper --run 
    --name "Virt-Viewer" 
    --outfile "Virt-Viewer-for-Windows.exe" 
    --with-gtk 
    /usr/i686-pc-mingw32/sys-root/mingw/bin/virt-viewer.exe
</pre>

缺少的dll从网上下或者windows环境中安装的virtviewer加到$PATH中  
这样会缺少三个xml文件，在virtviewer/src里，直接copy过去

<pre>export PATH=$PATH:/usr/i686-pc-mingw32/sys-root/mingw/bin:.
</pre>

windows client中使用usbredir：  
需要libwdi或者zadig或者usbclerk  
<a href="http://lists.freedesktop.org/archives/spice-devel/2012-November/011629.html" title="usbredir in windows client" target="_blank">链接</a>

todo:  
只有libvirt不是最新的了，可是目前来看没影响