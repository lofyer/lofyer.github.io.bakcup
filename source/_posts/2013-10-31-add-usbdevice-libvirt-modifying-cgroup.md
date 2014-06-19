---
title: Add usbdevice to libvirt by modifying cgroup
author: lofyer
layout: post
comments: true
permalink: /add-usbdevice-libvirt-modifying-cgroup/
categories:
  - Virtualization
---
CGROUP means Control group.  
In my case, usb-redir works fine while usb-host not.  
<a href="https://www.kernel.org/doc/Documentation/cgroups/devices.txt" title="Cgroup HowTO" target="_blank">https://www.kernel.org/doc/Documentation/cgroups/devices.txt</a>

## Just do it

<pre># sed -i 's/devices /#devices /'
# reboot
</pre>

## Add fine control list

### Review the rules to libvirt, and leave all the character device to it

<pre># cgget libvirt
To remove "c" device
# echo c > /sys/fs/cgroup/libvirt/devices.deny
To add all the "c" devices
# echo "c *:* rwm" > /sys/fs/cgroup/libvirt/devices.allow
</pre>

devices.deny/allow is something like end-point.

### About hierarchy

**A child will not have more permission than its parent.**