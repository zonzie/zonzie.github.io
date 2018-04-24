---
title: centos7_64在vmWare中的联网配置
date: 2018-04-24 17:19:54
tags: 
	- linux 
	- vmWare
categories: linux
description: linux在vmWare中的联网配置
---
- vmware使用NAT模式的网络配置
    1. 首先在虚拟机下的 编辑虚拟机设置->网络设配器->网络连接中的自定义->选择VMnet8(NAT模式)
    2. 虚拟机的选项卡 编辑->虚拟网络编辑器->添加网络->选择VMnet8(如果之前有就不能再次添加了,只能有一个)
    3. 添加后修改子网IP为centos7的ip地址,最后一位改为0,点击 NAT设置->设置网关为IP地址最后一位改为2
- 修改centos7的网卡配置:
    1. 进入网卡文件 `cd /etc/sysconfig/network-scripts/`
    2. 备份原有的网卡,我的是ifcfg-ens33 `cp ifcfg-ens33 ./ifcfg-ens33.bak`
    3. 修改原来的网卡,具体配置如下: `vi ifcfg-ens33`
```
    DEVICE=ens33
    TYPE=Ethernet
    # 静态ip static
    BOOTPROTO=static                  
    DEFROUTE=yes
    PEERDNS=yes
    NM_CONTROLLED=yes
    PEERROUTES=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=no
    NAME="system eth0"
    UUID=5df66116-fefc-4c67-84ec-3153c8d0d893
    ONBOOT=yes
    # ip地址
    IPADDR=192.168.198.88
    # 网关
    GATEWAY=192.168.198.2
    NETMASK=255.255.255.0
    # 需要配置一个DNS,不然无法解析域名
    DNS1=8.8.8.8
    USERCTL=no
```
    4. 修改完毕后,保存文件
    5. 重启网卡,依次执行以下命令
    `systemctl stop NetworkManager`
    ,`systemctl disable NetworkManager`
    ,`systemctl restart network`
    6. 查看网卡配置 `ip addr`
    7. 查看MAC地址: `vim /etc/udev/rules.d/70-persistent-net.rules`
    8. 网卡的MAC地址 ifcfg-eth* 要和上述的文件中的网卡地址保持一致