---
title: use MAXS to control your device via ejabberd(plus ssh, jingle voice talk as a bonus)
author: lofyer
layout: post
permalink: /ejabber-howto/
videourl:
  - 
categories:
  - 'IM-PBX'
---
Let&#8217;s see what we have got here:  
A xmpp server based on ejabberd on my host: lofyer.org.  
Windows client: <a href="https://download.jitsi.org/jitsi/nightly/" target="_blank">Jitsi(Recommended)</a>, <a href="http://www.pidgin.im/" target="_blank">Pidgin</a>.  
(Optional)A Android client: <a href="http://www.xabber.org/" target="_blank">Xabber.</a>  
<a href="http://f-droid.org/wiki/page/org.projectmaxs.main" target="_blank">MAXS</a> on my Nexus 5 Android phone.

## 1. Prepare the server(Debian 7)

<pre># apt-get install ejabberd
# cd /etc/ejabberd/; wget http://people.collabora.com/~robot101/olpc-ejabberd/ejabberd.cfg
</pre>

Change **hosts** and **admin** section to your **FQDN**.  
Here&#8217;s a example:

<pre>{hosts, ["lofyer.org"]}.
{acl, admin, {user, "mypassword", "lofyer.org"}}.
</pre>

Then you should restart ejabberd, and maybe a reboot is essential.

<pre># /etc/init.d/ejabberd restart
</pre>

### Enable Jingle(voice and video)

You need JingleNodes module on your server.

<pre># apt-get install erlang-tools
# git clone git://git.process-one.net/exmpp/mainline.git exmpp
# cd exmpp; ./configure; make; make install
# svn checkout http://jinglenodes.googlecode.com/svn/ jinglenodes
# cd jinglenodes; ./configure --prefix=/usr/; make; make install
</pre>

Add following content to your ejabberd.cfg in the **modules** section.

<pre>{mod_jinglenodes, [
		     {host, "jinglenodes.@HOST@"},
		     {public_ip, "192.168.1.148"},
		     {purge_period, 5000},
		     {relay_timeout, 60000}
		    ]},
</pre>

### Enable web register(optional)

Add to ejabberd.cfg, &#8216;modules&#8217; section the basic configuration:

<pre>{modules, [
  ...
  {mod_register_web,     []},
  ...
]}.
</pre>

In the &#8216;listen&#8217; section enable the web page:

<pre>{listen, [
  ...
  {5281, ejabberd_http, [
	tls,
    {certfile, "/etc/ejabberd/ejabberd.pem"},
    {request_handlers, [
      {["register"], mod_register_web}
    ]}
  ]},
  ...
]}.
</pre>

### Use your own certificate

<pre>openssl req -new -x509 -newkey rsa:1024 -days 3650 -keyout privkey.pem -out server.pem
openssl rsa -in privkey.pem -out privkey.pem
cat privkey.pem >> server.pem
rm privkey.pem
</pre>

The port numbers you should open are: 5281(http://localhost:5281/register/) 5280(http://localhost:5280/admin) and 5222(for c2s).

Register users:

<pre># ejabberdctl register admin lofyer.org mypassword
# ejabberdctl register myphone lofyer.org mypassword
# ejabberdctl register mypc lofyer.org mypassword
</pre>

## 2. Pidgin and MAXS test

Pidgin: admin@lofyer.org  
MAXS: myphone@lofyer.org  
By the way, guarantee that there is only one running jabber client on your phone during this period.  
**Pidgin**  
Add a friend  
<a href="http://blog.lofyer.org/ejabber-howto/pidgin/" rel="attachment wp-att-2822"><img src="http://blog.lofyer.org/wp-content/uploads/pidgin.png" alt="pidgin" width="266" height="443" class="alignnone size-full wp-image-2822" /></a>  
**Shell test**  
<a href="http://blog.lofyer.org/ejabber-howto/shell/" rel="attachment wp-att-2824"><img src="http://blog.lofyer.org/wp-content/uploads/shell.png" alt="shell" width="356" height="428" class="alignnone size-full wp-image-2824" /></a>  
**SMS test**  
<a href="http://blog.lofyer.org/ejabber-howto/sms-send/" rel="attachment wp-att-2825"><img src="http://blog.lofyer.org/wp-content/uploads/SMS-SEND.png" alt="SMS-SEND" width="546" height="393" class="alignnone size-full wp-image-2825" /></a>  
And a msg to my GF.  
<a href="http://blog.lofyer.org/ejabber-howto/sms-receive/" rel="attachment wp-att-2826"><img src="http://blog.lofyer.org/wp-content/uploads/sms-receive.jpg" alt="sms-receive" width="545" height="325" class="alignnone size-full wp-image-2826" /></a>
