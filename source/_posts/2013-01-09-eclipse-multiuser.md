---
title: eclipse multiuser
author: lofyer
layout: post
comments: true
categories:
  - Program
---
Add following code to your eclipse/configuration/config.ini

<pre>osgi.configuration.area=@user.home/Eclipse/configuration
osgi.sharedConfiguration.area=/opt/eclipse/configuration
osgi.configuration.cascaded=true
</pre>