---
title: 'use Foreman/Nagios/Icinga to make life easy&#8230;'
author: lofyer
layout: post
comments: true
permalink: /nagiosicinga-event-handler-life-easy/
categories:
  - Linux Admin
---
# Install nagios in Gentoo/CentOS

## Gentoo

<pre># emerge nagios</pre>

### Option: recompile apache for php support

add use flag &#8220;apache2&#8243; to /etc/portage/make.conf

<pre># emerge --ask --changed-use --deep @world</pre>

### Copy following content to /etc/apache2/vhosts.d/

<pre>ScriptAlias /nagios/cgi-bin "/usr/lib64/nagios/cgi-bin"
&lt;Directory "/usr/lib64/nagios/cgi-bin"&gt;
#  SSLRequireSSL
    Options ExecCGI
    AllowOverride None
    Order allow,deny
    Allow from all
#  Order deny,allow
#  Deny from all
#  Allow from 127.0.0.1
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /etc/nagios/auth.users
    Require valid-user

Alias /nagios "/usr/share/nagios/htdocs"
&lt;Directory "/usr/share/nagios/htdocs"&gt;
#  SSLRequireSSL
    Options None
    AllowOverride None
    Order allow,deny
    Allow from all
#  Order deny,allow
#  Deny from all
#  Allow from 127.0.0.1
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /etc/nagios/auth.users
    Require nagiosadmin</pre>

### Create password for nagiosadmin

<pre># htpasswd2 -c /etc/nagios/auth.users nagiosadmin</pre>

### Add NAGIOS to apache config

/etc/conf.d/apache

<pre>APACHE2_OPTS="... -D NAGIOS -D PHP5"</pre>

Add user nagios to apache group

<pre># usermod -a -G nagios apache</pre>

Start service

<pre># rc-service nagios restart
# rc-service apache2 restart</pre>

## CentOS

<pre># yum install "nagios*"
# htpasswd -c /etc/nagios/passwd admin
# chkconfig nagios on
# chkconfig httpd on
# service nagios start
# service httpd start
</pre>

## Add routers/hosts, add service, add hooks

## Intergrate with oVirt

# using Foreman

## Install

## USE

## Intergrate with oVirt

TBD