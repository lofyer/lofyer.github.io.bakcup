---
title: Migrate vm from ESiX to oVirt
author: lofyer
layout: post
comments: true
categories:
  - Virtualization
---
oVirt: v3.3, CentOS, 192.168.1.111  
ESiX: 5.x, 192.168.1.135

1. Create a nfs export domain on your oVirt datacenter.

<pre title="/etc/exports">/vdsm/export	0.0.0.0/0.0.0.0(rw)</pre>

2. Install virt-v2v on CentOS and add authentication.

<pre># yum install virt-v2v</pre>

<pre title="~/.netrc">machine 192.168.1.135 login root password 1234567</pre>

Change permission of *.netrc* as saying of manual or you will get a warning with wrong permission.

<pre># chmod 600 ~/.netrc</pre>

3. Import myvm. **Make sure that your vm in ESiX is powered off.**

<pre># virt-v2v -ic esx://192.168.1.135/?no_verify=1 -o rhev -os 192.168.1.111:/virtfan/export --network mgmtnet myvm
myvm_myvm: 100% [====================================================]D 0h04m48s

virt-v2v: myvm configured with virtio drivers.</pre>

4. Import myvm from export domain to data domain.

**Import.**

<a href="http://blog.lofyer.org/migrate-vm-esix-ovirt/import/" rel="attachment wp-att-3185"><img src="http://io.lofyer.org/uploads/import.png" alt="import" width="824" height="180" class="alignnone size-full wp-image-3185" /></a>

**Run.**

<a href="http://blog.lofyer.org/migrate-vm-esix-ovirt/run/" rel="attachment wp-att-3186"><img src="http://io.lofyer.org/uploads/run.png" alt="run" width="1065" height="569" class="alignnone size-full wp-image-3186" /></a>