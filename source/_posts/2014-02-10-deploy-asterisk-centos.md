---
title: Deploy Asterisk on CentOS
author: lofyer
layout: post
permalink: /deploy-asterisk-centos/
categories:
  - 'IM &amp; PBX'
---
Get the latest packages. **up to 2014-02-10**

<pre># rpm -Uvh http://packages.asterisk.org/centos/6/current/x86_64/RPMS/asterisknow-version-3.0.1-2_centos6.noarch.rpm
# yum install asterisk asterisk-configs --enablerepo=asterisk-12
# yum install dahdi-linux dahdi-tools libpri
# chkconfig dahdi on
# chkconfig asterisk on
# service dahdi start
# service asterisk start
</pre>

You can use freepbx on http://localhost .

<pre># yum install freepbx
</pre>