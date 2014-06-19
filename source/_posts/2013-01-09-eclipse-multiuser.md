---
title: eclipse multiuser
author: lofyer
layout: post
comments: true
permalink: /eclipse-multiuser/
categories:
  - Program
---
Add following code to your eclipse/configuration/config.ini

<pre>osgi.configuration.area=@user.home/Eclipse/configuration
osgi.sharedConfiguration.area=/opt/eclipse/configuration
osgi.configuration.cascaded=true
</pre>