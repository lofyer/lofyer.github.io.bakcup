---
title: 14. oVirt使用进阶
author: lofyer
layout: post
comments: true
videourl:
  - 
categories:
  - 云端日记
---
## ovirt-shell

使用ovirt-shell在一定程度上适合于某些场景。

<pre># ovirt-shell -I -u admin@internal -l https://server_ip/api

  ============================================================================
                      >>> connected to oVirt manager 3.4.0.0 &lt;&lt;&lt;
  ============================================================================

  ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

                             Welcome to oVirt shell

  ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++


[oVirt shell (connected)]#
</pre>

## 主机hooks

参考<a href="https://github.com/oVirt/vdsm/tree/master/vdsm_hooks" target="_blank">vdsm-hooks</a>。

## 集群策略

<a href="http://www.ovirt.org/images/2/2a/Scheduler-Deep-Dive-oVirt.pdf" target="_blank">参考这个PDF</a>。

## libguestfs扩容

**LVM扩容**  
**普通扩容**

## UI plugin

**ShellInABox**

## 平台插件

**Foreman**  
**OpenStack Network**  
**OpenStack Image**

## 手动创建导出域

构造目录，形如：

<pre># tree exports
.
├── 37e0e64b-5445-4bc3-8675-ceff4637e8e3/
│   ├── dom_md/
│   │   ├── ids
│   │   ├── inbox
│   │   ├── leases
│   │   ├── metadata
│   │   └── outbox
│   ├── images/
│   └── master/
│       ├── tasks/
│       └── vms/
└── __DIRECT_IO_TEST__</pre>

创建*leases*文件：

<pre title="create leases"># echo 2d2d2d2d2d2d465245452d2d2d2d2d2d3030303030303030303030303030
303000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000
0000 | xxd -r -p > leases
</pre>

添加如下内容至*metadata*文件：

<pre title="metadata">CLASS=Backup
DESCRIPTION=Export
IOOPTIMEOUTSEC=10
LEASERETRIES=3
LEASETIMESEC=60
LOCKPOLICY=
LOCKRENEWALINTERVALSEC=5
MASTER_VERSION=0
POOL_UUID=
REMOTE_PATH=192.168.1.108:/home/nfs_exports
ROLE=Regular
SDUUID=37e0e64b-5445-4bc3-8675-ceff4637e8e3
TYPE=NFS
VERSION=0
#_SHA_CKSUM=5737f1270bf93fdd660fea819655b01a34c315b9
</pre>

使用如下脚本（参考源码中packaging/setup/plugins/ovirt-engine-setup/config/iso_domain.py）计算SHA校验值，并将其填入*metadata*中的\_SHA\_CKSUM段：

<pre title="domain_chksum.py">#!/usr/bin/python

import hashlib
from optparse import OptionParser

if __name__  == "__main__":
    parser = OptionParser()
    (options, args) = parser.parse_args()
    if len(args) != 1:
        parser.error("Missing metadata file")
    f = open(args[0], "r")
    mds = {}
    for line in f:
        line = line.strip()
        if not line or line.startswith('#'):
            continue
        try:
            key,value = line.split('=', 1)
            if key == '_SHA_CKSUM':
                continue
            mds[key] = value
        except Exception, e:
            continue
    f.close()
    #print mds
    csum = hashlib.sha1()
    keys = mds.keys()
    keys.sort()
    for key in keys:
        value = mds[key]
        line = "%s=%s" % (key, value)
        csum.update(line)
    print(csum.hexdigest())
</pre>

更改权限：

<pre># chown -R vdsm.kvm exports</pre>

然后可以作为空导出域进行导入。