---
title: VMware CentOS7 静态IP配置
date: 2019-07-27 19:09:03
categories: 
- Linux
tags:
- linux
- centos
- VMware
description: 学习linux可以在windows中使用VMware安装CentOS操作系统，使用Xshell远程访问VMware中的CentOS操作系统，由于每次重启CentOS IP都可能变化，因此需要对VMware中 CentOS设置静态IP。
---

1.点击左上角编辑选择虚拟网络编辑器  
{% asset_img '选择虚拟网络编辑器.png' 选择虚拟网络编辑器 %}

2. 选择NAT模式，点击更改设置
{% asset_img '选择NAT模式.png' 选择NAT模式 %}

3. 选择NAT模式，取消使用本地DHCP服务将IP地址分配给虚拟机(可选)，点击NAT设置查看子网IP、子网掩码、网关IP,
{% asset_img '查看NAT设置.png' 查看NAT设置 %}

4. 查看网卡信息
```
[root@centos7 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:59:3e:28 brd ff:ff:ff:ff:ff:ff
    inet 192.168.189.128/24 brd 192.168.189.255 scope global noprefixroute dynamic ens33
       valid_lft 1650sec preferred_lft 1650sec
    inet6 fe80::e1e8:75df:33ef:8461/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

5. 编辑网卡信息  
网卡名称为ens33,该网卡的配置文件在/etc/sysconfig/network-scripts/ifcfg-ens33  
主要修改BOOTPROTO参数的值为static,添加IPADDR、GATEWAY、NETMASK、DNS1四个参数。
IPADDR、GATEWAY、NETMASK应和截图中⑧保持一致。
```
[root@centos7 ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33 
[root@centos7 ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"      #使用静态IP地址,默认为dhcp
IPADDR=192.168.189.131  #IP地址
GATEWAY=192.168.189.2   #网关地址
NETMASK=255.255.255.0   #子网掩码
DNS1=114.114.114.114    #DNS地址
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="f49c4e07-52c1-45bc-a58c-e75894e1638c"
DEVICE="ens33"
ONBOOT="yes"
```

6. 重启网络服务  
使用service network restart重启网络服务，由于使用远程连接虚拟机内CentOS，因此连接被断开。
```
[root@centos7 ~]# service network restart
Restarting network (via systemctl):  
Socket error Event: 32 Error: 10053.
Connection closing...Socket close.

Connection closed by foreign host.

Disconnected from remote host(localhost_CentOS7) at 18:35:03.
```
7. 检查网络是否配置生效  
再次查看网卡信息如下,发现IP已经发生变化
```
[root@centos7 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:59:3e:28 brd ff:ff:ff:ff:ff:ff
    inet 192.168.189.131/24 brd 192.168.189.255 scope global noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::e1e8:75df:33ef:8461/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
8. 检查网络是否畅通  
使用ping命令检查是否能够访问 baidu.com ,有数据返回，说明配置无误。
```
[root@centos7 ~]# ping baidu.com
PING baidu.com (39.156.69.79) 56(84) bytes of data.
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=1 ttl=128 time=61.5 ms
64 bytes from 39.156.69.79 (39.156.69.79): icmp_seq=2 ttl=128 time=61.5 ms
^C
--- baidu.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 61.564/61.573/61.582/0.009 ms
```
