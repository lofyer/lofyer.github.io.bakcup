---
title: Add nat to ovirt via vdsm hooks
author: lofyer
layout: post
permalink: /add-nat-ovirt-vdsm-hooks/
videourl:
  - 
categories:
  - Virtualization
---
OK, I do not like control group very much for now.

## Comment device section in control group config, sorry for my laziness&#8230;

<pre title="/etc/cgconfig.conf">mount {
        cpuset  = /cgroup/cpuset;
        cpu     = /cgroup/cpu;
        cpuacct = /cgroup/cpuacct;
        memory  = /cgroup/memory;
#       devices = /cgroup/devices;
        freezer = /cgroup/freezer;
#       net_cls = /cgroup/net_cls;
        blkio   = /cgroup/blkio;
}
</pre>

## Enable ip forward

<pre title="Content added to /etc/sysctl.conf">net.ipv4.ip_forward = 1
</pre>

**Reboot the host**

## Connect to virsh to enable libvirt&#8217;s default virbr0(NAT)

If you don&#8217;t know the password for your account, just use command below to create one.

<pre># saslpasswd2 -a libvirt root
</pre>

Create a nat network.

<pre title="/etc/libvirt/qemu/networks/nat.xml" lang="xml">&lt;network>
  &lt;name>nat&lt;/name>
  &lt;uuid>b42e377d-e849-4c36-bd98-3d090def5ecc&lt;/uuid>
  &lt;bridge name="virbr1"/>
  &lt;mac address='52:54:00:E2:AD:b8'/>
  &lt;forward mode="nat"/>
  &lt;ip address="192.168.123.1" netmask="255.255.255.0">
    &lt;dhcp>
      &lt;range start="192.168.123.2" end="192.168.123.254"/>
    &lt;/dhcp>
  &lt;/ip>
&lt;/network>
</pre>

<pre># virsh net-create /etc/libvirt/qemu/networks/nat.xml
# virsh net-autostart nat
# virsh net-start nat
</pre>

## Create tun device and add it to virbr0

**UPDATE: This could be ignored if you use extnet.py**

<pre># tunctl -t nat0 -u qemu
# brctl addif virbr1 nat0
</pre>

## Add hook file to vdsm

**UPDATE: use extnet from github with a little modification (Only the first vNIC will be NAT, the second one still keeps its way).**

<pre title="extnet.py" lang="python">#!/usr/bin/python

import os
import sys
import traceback
import xml.dom
import hooking

def replaceSource(interface, newnet):
    source, = interface.getElementsByTagName('source')
    source.removeAttribute('bridge')
    source.setAttribute('network', newnet)
    interface.setAttribute('type', 'network')

def main():
    params = "default"
    os.environ.__setitem__("extnet",params)
    newnet = os.environ.get('extnet')
    if newnet is not None:
        doc = hooking.read_domxml()
        interface = doc.getElementsByTagName('interface')[0]
        replaceSource(interface, newnet)
        hooking.write_domxml(doc)
def test():

    interface = xml.dom.minidom.parseString("""
    &lt;interface type="bridge">
        &lt;address bus="0x00" domain="0x0000" function="0x0" slot="0x03"\
                                            type="pci"/>
        &lt;mac address="00:1a:4a:16:01:b0"/>
        &lt;model type="virtio"/>
        &lt;source bridge="ovirtmgmt"/>
        &lt;filterref filter="vdsm-no-mac-spoofing"/>
        <link state="up" />
&lt;boot order="1"/>
    &lt;/interface>
    """).getElementsByTagName('interface')[0]

    print "Interface before forcing network: %s" % \
        interface.toxml(encoding='UTF-8')

    replaceSource(interface, 'yipee')
    print "Interface after forcing network: %s" % \
        interface.toxml(encoding='UTF-8')


if __name__ == '__main__':
    try:
        if '--test' in sys.argv:
            test()
        else:
            main()
    except:
        hooking.exit_hook('extnet hook: [unexpected error]: %s\n' %
                          traceback.format_exc())
</pre>

**QEMU-CMD way:**

<pre lang="python" title="/usr/libexec/vdsm/hooks/before_vm_start/nat.py with execute permission">import os
import sys
import hooking
import traceback
import json
import shutil

def addQemuNs(domXML):
    domain = domXML.getElementsByTagName('domain')[0]
    domain.setAttribute('xmlns:qemu',
                        'http://libvirt.org/schemas/domain/qemu/1.0')


def injectQemuCmdLine(domXML, qc):
    domain = domXML.getElementsByTagName('domain')[0]
    qctag = domXML.createElement('qemu:commandline')

    for cmd in qc:
        qatag = domXML.createElement('qemu:arg')
        qatag.setAttribute('value', cmd)

        qctag.appendChild(qatag)

    domain.appendChild(qctag)
domxml = hooking.read_domxml()

# Get vm uuid, just in case

cur_vm_uuid = domxml.getElementsByTagName('uuid')[0].firstChild.nodeValue

macaddr = "94:de:80:ea:30:f5"
natname = "nat0"
params = '["-netdev","tap,ifname=%s,script=no,id=hostnet0,downscript=no","-device","virtio-net-pci,mac=%s,netdev=hostnet0,bus=pci.0,addr=0x10"]' % (natname, macaddr)
os.environ.__setitem__("qemu_cmdline",params)

# Modify Qemu Parameter

if 'qemu_cmdline' in os.environ:
    try:
        domxml = hooking.read_domxml()

        qemu_cmdline = json.loads(os.environ['qemu_cmdline'])
        addQemuNs(domxml)
        injectQemuCmdLine(domxml, qemu_cmdline)

        hooking.write_domxml(domxml)
    except:
        sys.stderr.write('qemu_cmdline: [unexpected error]: %s\n'
                         % traceback.format_exc())
        sys.exit(2)
</pre>

Then you should start the vm **WITHOUT ANY NIC** if you are using *nat.py*.