---
title: Intergrate owncloud with AD(LDAP)
author: lofyer
layout: post
comments: true
permalink: /intergrate-owncloud-adldap/
categories:
  - LDAP
  - OwnCloud
---
Windows 2008R2 server with AD role built.  
User group: **owncloudgrp**  
User in **owncloudgrp**: aaa, beta  
Users must have **logon name, first name, last name** set.

## Configure the owncloud:

Server:

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc1/" rel="attachment wp-att-3165"><img src="http://blog.lofyer.org/wp-content/uploads/oc1.png" alt="oc1" width="833" height="262" class="alignnone size-full wp-image-3165" /></a>

User Filter:

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc2/" rel="attachment wp-att-3166"><img src="http://blog.lofyer.org/wp-content/uploads/oc2.png" alt="oc2" width="834" height="354" class="alignnone size-full wp-image-3166" /></a>

Login Filter:

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc3/" rel="attachment wp-att-3167"><img src="http://blog.lofyer.org/wp-content/uploads/oc3.png" alt="oc3" width="850" height="260" class="alignnone size-full wp-image-3167" /></a>

Group Filter:  
Every time you change these two sections, **wait for a few seconds** until more than zero users discovered.

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc4/" rel="attachment wp-att-3168"><img src="http://blog.lofyer.org/wp-content/uploads/oc4.png" alt="oc4" width="844" height="364" class="alignnone size-full wp-image-3168" /></a>

Advanced &#8211; Connection Settings:

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc5/" rel="attachment wp-att-3169"><img src="http://blog.lofyer.org/wp-content/uploads/oc5.png" alt="oc5" width="842" height="409" class="alignnone size-full wp-image-3169" /></a>

Advanced &#8211; Directory Settings:

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc6/" rel="attachment wp-att-3170"><img src="http://blog.lofyer.org/wp-content/uploads/oc6.png" alt="oc6" width="904" height="545" class="alignnone size-full wp-image-3170" /></a>

Expert:  
Add internal username: sAMAccountName

<a href="http://blog.lofyer.org/intergrate-owncloud-adldap/oc7/" rel="attachment wp-att-3171"><img src="http://blog.lofyer.org/wp-content/uploads/oc7.png" alt="oc7" width="893" height="399" class="alignnone size-full wp-image-3171" /></a>