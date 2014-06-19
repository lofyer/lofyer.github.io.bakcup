---
title: Openstack 配置及其API的简单使用
author: lofyer
layout: post
comments: true
permalink: /openstack-configure-and-api/
categories:
  - Virtualization
---
## 安装除Neutron（Quantum）以外所有东西

在开始之前，假设有三个NIC<pre chkconfig iptables off chkconfig firewalld off chkconfig sshd on chkconfig NetworkManager off chkconfig network start yum install mysql-server service mysqld start mysqladmin -uroot password 123456 chkconfig mysqld on </pre> 

禁用selinux，并修改网络配置/etc/sysconfig/network-scripts/ifcfg-ethX

<pre># Internal
DEVICE=eth0
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.11
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1
DEFROUTE=yes
ONBOOT=yes
</pre>

<pre># External
DEVICE=eth1
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.12
NETMASK=255.255.255.0
DEFROUTE=yes
ONBOOT=yes
</pre>

<pre>#Public Bridge
DEVICE=eth2
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.13
NETMASK=255.255.255.0
DEFROUTE=yes
ONBOOT=yes
</pre>

于Feodra18中安装Openstack，可参考文档，按照这个来的话对Openstack的设计理念将会有所了解  
<a href="https://fedoraproject.org/wiki/Getting_started_with_OpenStack_on_Fedora_18" title="Getting started with OpenStack on Fedora 18" target="_blank">Getting started with OpenStack on Fedora 18</a>  
或者运行

<pre>/*安装完成后管理员界面为http://localhost/dashboard用户名admin，密码secrete*/
# openstack-demo-install
</pre>

以上两种方法把除了Neutron（quantum）其他的服务都配置好了，关于quantum的配置见文末

## 添加虚拟机实例

可参考文档  
<a href="https://fedoraproject.org/wiki/Getting_started_with_OpenStack_on_Fedora_18" title="Getting started with OpenStack on Fedora 18" target="_blank">Getting started with OpenStack on Fedora 18</a>  
在终端操作之前需要设置环境变量，将它们两者之一写到一个文件，然后source一下

<pre>export OS_USERNAME=admin
export OS_PASSWORD=secrete
export OS_TENANT_NAME=admin
export OS_AUTH_URL=http://127.0.0.1:5000/v2.0/
</pre>

<pre>export ADMIN_TOKEN=$(openssl rand -hex 10)
export SERVICE_ENDPOINT=http://127.0.0.1:35357/v2.0/
export SERVICE_TOKEN=\$ADMIN_TOKEN
</pre>

<pre>source FileA
</pre>

这个libvirt支持kvm以及xen虚拟化，所以支持的镜像也比较多，这里下载一个已经安装好的f17的qcow2镜像

<pre># axel -n 5 http://berrange.fedorapeople.org/images/2012-11-15/f17-x86_64-openstack-sda.qcow2
/*注册镜像*/
# glance add name=f17-jeos is_public=true disk_format=qcow2 container_format=bare &lt; f17-x86_64-openstack-sda.qcow2
</pre>

然后以demo用户进入dashboard，会看到添加进来的image，根据image启动实例，打开vnc界面（非localhost访问要改ip）。

## 安装操蛋的Neutron（quantum）服务

这东西刚改名，nova自带的有network服务，可这个可以虚拟L2 L3 switch，那就比较有意思了

/\*注册quantum\*/

<pre># keystone service-create --name quantum --type network --description 'OpenStack Networking Service'
# keystone endpoint-create --region myregion --service-id 26a55b340e254ad5bb78c0b14391e153 --publicurl "http://192.168.1.11:9696/" --adminurl "http://192.168.1.11:9696/" --internalurl "http://192.168.1.11:9696/"
</pre>

/\*(可选)添加quantum服务用户\*/  
get_id.sh:

<pre>$ function get_id () {
     echo `"$@" | awk '/ id / { print $4 }'`
}
# ADMIN_ROLE=$(get_id keystone role-create --name=admin)
# QUANTUM_USER=$(get_id keystone user-create --name=quantum --pass="servicepass" --email=demo@example.com --tenant-id service)
# keystone user-role-add --user_id $QUANTUM_USER --role_id $ADMIN_ROLE --tenant_id service
</pre>

/\*服务插件二选一，linuxbrideg和openvswitch，这里使用后者，先安装插件\*/

<pre># yum install openstack-quantum-openvswitch
</pre>

/\*启动并添加服务\*/

<pre># quantum-server-setup --plugin openvswitch
# quantum-node-setup --plugin openvswitch
# service quantum-server start
# chkconfig quantum-server on

# service openvswitch start
# chkconfig openvswitch on

# ovs-vsctl add-br br-int
# chkconfig quantum-openvswitch-agent on
# service quantum-openvswitch-agent start

# quantum-dhcp-setup --plugin openvswitch
# chkconfig quantum-dhcp-agent on
# service quantum-dhcp-agent start

# ovs-vsctl add-br br-ex
# ovs-vsctl add-port br-ex eth1
# quantum-l3-setup --plugin openvswitch
# chkconfig quantum-l3-agent on
# service quantum-l3-agent start
<del datetime="2013-07-18T07:31:22+00:00"># chkconfig quantum-metadata-agent on
# service quantum-metadata-agent start</del>
</pre>

修改并配置网络 /etc/sysconf/network-scripts/ifcfg-eth1，/etc/sysconf/network-scripts/ifcfg-br-ex

<pre># External
DEVICE=eth1
TYPE=Ethernet
BOOTPROTO=none
NM_CONTROLLED=no
BRIDGE=br-ex
ONBOOT=yes
</pre>

<pre>#Public Bridge
DEVICE=br-ex
TYPE=Bridge
BOOTPROTO=static
IPADDR=192.168.2.11
NETMASK=255.255.255.0
NM_CONTROLLED=no
ONBOOT=yes
</pre>

<pre># ip addr del 10.0.0.9/24 dev eth1
# ip addr add 10.0.0.9/24 dev br-ex
</pre>

这里类似VPN的重写HEADER

<pre>echo 1 > /proc/sys/net/ipv4/conf/all/forwarding
sysctl -w net.ipv4.ip_forward=1 
iptables -A FORWARD -i eth0 -o br-ex -s 10.10.10.0/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A POSTROUTING -s 10.10.10.0/24 -t nat -j MASQUERADE
</pre>

最后重启机器即可

## 通过curl使用openstack api

<a href="http://catn.com/2013/04/23/create-an-openstack-instance-with-just-curl/" title="http://catn.com/2013/04/23/create-an-openstack-instance-with-just-curl/" target="_blank">参考这篇文章</a>  
注意区分token和固定id