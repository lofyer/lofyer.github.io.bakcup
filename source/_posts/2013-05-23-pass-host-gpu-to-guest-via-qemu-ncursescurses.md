---
title: GPU Passthrough, VGA Passthrough in KVM
author: lofyer
layout: post
comments: true
permalink: /pass-host-gpu-to-guest-via-qemu-ncursescurses/
duoshuo_thread_id:
  - 1234836220387786875
enclosure:
  - |
    |
        http://blog.lofyer.org/wp-content/uploads/Arch-Linux-KVM-Crysis-HD.flv
        27238793
        video/x-flv
        
videourl:
  - 
categories:
  - Linux Admin
---
To inspire you, I&#8217;ve got a video from someone else. Better mutt the volume by the way.  
<a href="http://www.youtube.com/watch?v=Qi1LdFkRzIs" title="Arch Linux KVM Crysis HD Gpu Passthrough" target="_blank">Arch Linux KVM Crysis HD Gpu Passthrough</a>  
Or you can download it to see.  
<a href="http://blog.lofyer.org/wp-content/uploads/Arch-Linux-KVM-Crysis-HD.flv" title="Download the video" target="_blank">Download the video in HD</a>  
<embed src="http://player.youku.com/player.php/sid/XNTg1MzUwNjY4/v.swf" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash">
</embed>

Here&#8217;s the links I refer to:  
<a href="http://thread.gmane.org/gmane.comp.emulators.kvm.devel/71981" title="http://thread.gmane.org/gmane.comp.emulators.kvm.devel/71981" target="_blank">http://thread.gmane.org/gmane.comp.emulators.kvm.devel/71981</a>  
<a href="https://bbs.archlinux.org/viewtopic.php?id=162768" title="https://bbs.archlinux.org/viewtopic.php?id=162768" target="_blank">https://bbs.archlinux.org/viewtopic.php?id=162768</a>  
<a href="https://docs.google.com/document/d/1ef_nfl652L0HLn_wGvnpgjsBJd9LZzaV_-rIcEEoK8Y/edit?pli=1" title="https://docs.google.com/document/d/1ef_nfl652L0HLn_wGvnpgjsBJd9LZzaV_-rIcEEoK8Y/edit?pli=1" target="_blank">https://docs.google.com/document/d/1ef_nfl652L0HLn_wGvnpgjsBJd9LZzaV_-rIcEEoK8Y/edit?pli=1</a>  
<a href="http://www.linux-kvm.org/page/VGA_device_assignment" title="http://www.linux-kvm.org/page/VGA_device_assignment" target="_blank">http://www.linux-kvm.org/page/VGA_device_assignment</a>  
<a href="http://www.linux-kvm.org/page/VGA_device_assignment" title="http://www.linux-kvm.org/page/VGA_device_assignment" target="_blank">http://www.linux-kvm.org/page/How_to_assign_devices_with_VT-d_in_KVM</a>

## Result:  
VGAPassthrough: success in host F19, guest Windows7  
GPUPassthrough: success in Fedora-Rawhide

HOST:  
CPU: Core i5 3470  
GPU: ATI HD Radeon 7850  
OS: Fedora-Rawhide  
QEMU: qemu-1.5.1  
<a href="http://blog.lofyer.org/2013/05/pass-host-gpu-to-guest-via-qemu-ncursescurses/kvm-vgapassthrough/" rel="attachment wp-att-2380"><img src="http://blog.lofyer.org/wp-content/uploads/kvm-vgapassthrough.png" alt="kvm-vgapassthrough" width="614" height="360" class="alignnone size-full wp-image-2380" /></a>  
So, here&#8217;s the steps

## 0. Enable the mainboard VxT, iommu and alter the video device to Intel HD

## 1. See what we have got now.

<pre>lspci;lspci -n
</pre>

We have output below

<pre>...
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Pitcairn PRO [Radeon HD 7850]
01:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Cape Verde/Pitcairn HDMI Audio [Radeon HD 7700/7800 Series]
...
</pre>

<pre>...
01:00.0 0300: 1002:6819
01:00.1 0403: 1002:aab0
...
</pre>

You can see the pci bus and vendor.

## 2. Modify the kernel parameter, morprobe.d and libvirt.conf

Add follow parameters to grub.conf

<pre>intel_iommu=on pci-stub.ids=1002:6819,1002:aab0,vfio_iommu_type1.allow_unsafe_interrupts=1
</pre>

NOTE: If you have got an AMD cpu, please replace &#8220;interl_iommu=on&#8221; with &#8220;iommu=pt iommu=1&#8243;  
Add modprobe.conf to /etc/modprobe.d/ with this content:

<pre>blacklist radeon
options kvm ignore_msrs=1
options kvm allow_unsafe_interrupts=1
options kvm-amd npt=0
options kvm_intel emulate_invalid_guest_state=0
options vfio_iommu_type1 allow_unsafe_interrupts=1
</pre>

change the following options in /etc/libvirt/qemu.conf:

<pre># The user ID for QEMU processes run by the system instance.
user = "root"

# The group ID for QEMU processes run by the system instance.
group = "root"

......

# If clear_emulator_capabilities is enabled, libvirt will drop all
# privileged capabilities of the QEmu/KVM emulator. This is enabled by
# default.
#
# Warning: Disabling this option means that a compromised guest can
# exploit the privileges and possibly do damage to the host.
#
clear_emulator_capabilities = 0
</pre>

Reboot.

## 3. Using scripts below

### Version 1: VFIO-Passthrough

File: vfio-bind

<pre>#!/bin/bash
modprobe vfio-pci
for var in "$@"; do
        for dev in $(ls /sys/bus/pci/devices/$var/iommu_group/devices); do
                vendor=$(cat /sys/bus/pci/devices/$dev/vendor)
                device=$(cat /sys/bus/pci/devices/$dev/device)
                if [ -e /sys/bus/pci/devices/$dev/driver ]; then
                        echo $dev > /sys/bus/pci/devices/$dev/driver/unbind
                        fi
                echo $vendor $device > /sys/bus/pci/drivers/vfio-pci/new_id
        done
done
</pre>

Bind the device

<pre>./vfio-bind 0000:01:00.0 0000:01:00.1
</pre>

Start VM

<pre>#!/bin/bash
sudo modprobe vfio-pci

sudo qemu-system-x86_64 -no-user-config -nodefaults -m 2048M -smp 4 -boot menu=on \
-net nic -net user -enable-kvm -monitor stdio -vga qxl -global qxl-vga.vram_size=67108864 \
-spice port=6000,ipv4,disable-ticketing \
-device intel-hda,id=sound0,bus=pcie.0,addr=0x4 -device hda-duplex,id=sound0-codec0,bus=sound0.0,cad=0 \
-drive file=Windows7.iso,if=none,id=drive-ide0-1-0,readonly=on,format=raw -device ide-cd,bus=ide.1,unit=0,drive=drive-ide0-1-0,id=ide0-1-0 \
-drive file=/home/lofyer/gpu_passthrough/f17.qcow2,if=none,id=drive-virtio-disk0,format=qcow2,cache=none,werror=stop,rerror=stop,aio=threads -device virtio-blk-pci,scsi=off,bus=pcie.0,addr=0x7,drive=drive-virtio-disk0,id=virtio-disk0,bootindex=1 \
-device virtio-balloon-pci,id=balloon0,bus=pcie.0,addr=0x8 \
-M q35 \
-device piix4-ide,bus=pcie.0 \
-device ioh3420,bus=pcie.0,addr=1c.0,multifunction=on,port=1,chassis=1,id=root.1 \
-device vfio-pci,host=01:00.0,bus=root.1,addr=00.0,multifunction=on,x-vga=on \
-device vfio-pci,host=01:00.1,bus=root.1,addr=00.1 \
-fda virtio.vfd
</pre>

### Version 2: PCI-Passthrough

Bind device

<pre>#!/bin/bash
modprobe pci-stub
for id in 6819 aab0; do
    echo 1002 $id > /sys/bus/pci/drivers/pci-stub/new_id
done
for pci in 0000:01:00.{0,1}; do
    echo $pci > "/sys/bus/pci/devices/$pci/driver/unbind"
    echo $pci > /sys/bus/pci/drivers/pci-stub/bind
done
</pre>

Start VM

<pre>#!/bin/bash
qemu-system-x86_64 \
-hda ../f17.qcow2 \
-cdrom /run/media/lofyer/Cache/OS_ISO/cn_windows_7_ultimate_with_sp1_x64_dvd_u_677408.iso \
-m 2048 -balloon virtio -smp 4 -enable-kvm \
-device pci-assign,host=01:00.0
</pre>