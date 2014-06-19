---
title: Gpu Passthrough in Xen
author: lofyer
layout: post
comments: true
categories:
  - Virtualization
---
Considering that Xen&#8217;s gpu passthrough is about one year ahead of kvm, so let us try that.  
The process is something like kvm one as I wrote before.  
<a href="http://blog.lofyer.org/2013/05/pass-host-gpu-to-guest-via-qemu-ncursescurses/" title="KVM GPU-Passthrough, VGA-Passthrough" target="_blank">KVM GPU-Passthrough, VGA-Passthrough</a>

## Result:  
I failed in trying passthrough the graphic card to Win7 or XP.  
Now I&#8217;m try some patchs and re-compile Xen with my graphic card bios.bin.  


HOST:  
CPU: i5-3470  
GPU: ATI Radeon HD 7850  
OS: openSuse 12.3 KDE

## 0. Disable Radeon Driver

<pre># echo "blacklist radeon" >> /etc/modprobe.d/modprobe.conf
# mkinitrd
</pre>

## 1. Install Xen-utils

Install Xen and its utilities via Yast.

## 2. Modify grub.cfg

<pre>...
 echo    'Loading Xen 4.3.0_06-1.3 ...'
        multiboot       /xen-4.3.0_06-1.3.gz placeholder iommu=1 
        echo    'Loading Linux 3.10.1-3.g0cd5432-xen ...'
        module  /vmlinuz-3.10.1-3.g0cd5432-xen placeholder root=UUID=da6ac064-2a5e-48c7-8834-307cbcb551c3 ro   resume=/dev/disk/by-id/ata-ST500DM002-1BD142_Z3THR3V3-part3 splash=silent quiet showopts xen-pciback.hide=(00:00.0)(01:00.1)
        echo    'Loading initial ramdisk ...'
        module  /initrd-3.10.1-3.g0cd5432-xen
...
</pre>

Then reboot the host.

## 3. Bind the device

### Using Scripts

File: bind_lib.sh

<pre>#!/bin/bash
   #
   # License: GPLv2
   # Author: Peter Maloney
   #
   # Script to bind devices to pciback (or pci-stub)
   
   find_new_id() {
       device="$1"
       len=${#device}
       if [ "$len" -eq 12 ]; then
           device="${device:5:12}"
       fi
       lspci -n | grep "${device}" | cut -d' ' -f3 | sed -r "s/:/ /"
   }
   
   bindstub() {
       device="$1"
       echo "binddevice $device"
       
       if [ ! -e "/sys/bus/pci/devices/$device" ]; then
           echo "    ERROR: Device does not exist... cancelling"
           return
       fi
       
       # with pci-stub, you do new_id, then unbind, then bind
       
       echo "create new_id"
       chmod +w /sys/bus/pci/drivers/pci-stub/new_id
       new_id="$(find_new_id "$device")"
       echo "    echo \"$new_id\" > /sys/bus/pci/drivers/pci-stub/new_id"
       echo "$new_id" > /sys/bus/pci/drivers/pci-stub/new_id
       
       echo "unbind"
       if [ ! -e "/sys/bus/pci/devices/$device/driver" ]; then
           echo "    no driver to unbind"
       else
           chmod +w "/sys/bus/pci/devices/${device}/driver/unbind"
           echo "    echo -n \"$device\" > \"/sys/bus/pci/devices/$device/driver/unbind\""
           echo -n "$device" > "/sys/bus/pci/devices/$device/driver/unbind"
       fi
       
       echo "bind"
       chmod +w /sys/bus/pci/drivers/pci-stub/bind
       echo "    echo -n \"$device\" > /sys/bus/pci/drivers/pci-stub/bind"
       echo -n "$device" > /sys/bus/pci/drivers/pci-stub/bind
       
       echo
   }
   
   bindback() {
       device="$1"
       echo "binddevice $device"
       
       if [ ! -e "/sys/bus/pci/devices/$device" ]; then
           echo "ERROR: Device does not exist... cancelling"
           return
       fi
       
       # with pciback, you do unbind, then new_slot, then bind
       
       echo "unbind"
       if [ ! -e "/sys/bus/pci/devices/$device/driver" ]; then
           echo "    no driver to unbind"
       else
           chmod +w "/sys/bus/pci/devices/${device}/driver/unbind"
           echo "    echo -n \"$device\" > \"/sys/bus/pci/devices/$device/driver/unbind\""
           echo -n "$device" > "/sys/bus/pci/devices/$device/driver/unbind"
       fi
       
       echo "create new_slot"
       chmod +w /sys/bus/pci/drivers/pciback/new_slot
       echo "    echo -n \"$device\" > /sys/bus/pci/drivers/pciback/new_slot"
       echo -n "$device" > /sys/bus/pci/drivers/pciback/new_slot
       
       echo "bind"
       chmod +w /sys/bus/pci/drivers/pciback/bind
       echo "    echo -n \"$device\" > /sys/bus/pci/drivers/pciback/bind"
       echo -n "$device" > /sys/bus/pci/drivers/pciback/bind
       
       echo
   }
</pre>

<pre># source bind_lib.sh
# modprobe xen-pciback
# bindback "0000:01:00.0"
# bindback "0000:01:00.1"
</pre>

### Using xen-tools

<pre># xl pci-assignable-add 01:00.0
# xl pci-assignable-add 01:00.1
# xl pci-attach Domain-0 01:00.0
# xl pci-attach Domain-0 01:00.1
</pre>

## 4. Create VM

File: win7

<pre>name="win77"
description="None"
uuid="db64c1a6-1f84-4eff-48df-53c4b03b389e"
memory=2048
maxmem=2048
vcpus=4
on_poweroff="destroy"
on_reboot="destroy"
on_crash="destroy"
localtime=1

builder="hvm"
device_model="/usr/lib/xen/bin/qemu-dm"
kernel="/usr/lib/xen/boot/hvmloader"
boot="dc"
disk=[ 'file:/var/lib/xen/images/windowsvistax64/disk0.raw,hda,w','file:/home/lofyer/Driver.iso,hdc:cdrom,r' ]
#vif=[ 'bridge=virbr0, mac=00:12:3e:6c:2f:02, model=rtl8139' ]

stdvga=1
vnc=1
vnclisten="0.0.0.0"
vncconsole=0
#vncunused=1
soundhw="all"

monitor=1
viridian=1
sdl=0
#usb=1
acpi=1
apic=1
pae=1

#usbdevice='tablet'

serial="pty"
# Add by lofyer
device_model_version = "qemu-xen-traditional"
gfx_passthru=0
pci_power_mgmt=1
xen_platform_pci=1
pci_msitranslate=1
hap=1
pci = [ '01:00.0' , '01:00.1'  ]
</pre>

\# xl create win7  
\# remote-viewer vnc://localhost:5900

Then you should see Windows boot logo in vnc for few seconds, after that it will switch to the second monitor that attached to your ATI card.

### gfx_gpupassthru

As Xen wiki said:

> When you specify &#8220;gfx_passthru=1&#8243; the passthru graphics card will be made the primary graphics card in the VM, and the Xen Qemu-dm emulated Cirrus graphics card is disabled.  
> If you use &#8220;gfx\_passthru=0&#8243;, or don&#8217;t have gfx\_passthru= option at all, then the Xen Qemu-dm emulated Cirrus graphics card will be the primary in the VM, and the passthru graphics card will be secondary.

So, alter gfx_passthru to 1.