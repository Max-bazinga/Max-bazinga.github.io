---
title: nfs构建
date: 2017-02-21 08:31:34
tags: [nfs]
---
1. NFS (Network File Service)

    NFS用于Linux系统之间的文件共享

    (1) 实验环境，主机Ubuntu 9.04，VMware 6.5，虚拟机Ubuntu 9.04

    (2) 在主机上安装nfs服务软件，因为Ubuntu默认是没有安装的

    > $sudo aptitude install nfs-kernel-server

    或使用新立德包管理器安装

    (3) 在虚拟机上安装nfs客户端

    $sudo apt-get install nfs-common

    (4) 假设虚拟机使用的是桥接，IP地址为10.1.60.34即和主机在一个网段内。

    (5) 修改nfs配置文件/etc/exports，添加如下一行

    > /home/yourname/sharedir 10.1.60.34(rw,sync,no_root_squash)

    第一个参数是你要让客户机访问的目录，第二个是你允许的主机IP，最后的（）内是访问控制方式。

    (6) 注意，上面的主机IP不能使用＊来通配，否则在客户机上会出现访问拒绝，但是如果我们要设置局域网访问呢？怎么办，使用子网掩码例如：10.1.60.0/255.255.254.0即可让10.1.60.*和10.1.61.*都可以访问,还可以使用10.1.60/23这种方式类确定子网。

    (7) 在主机上启动NFS服务

    测试配置文件

    > $ sudo  exportfs  -r

    > $sudo /etc/init.d/portmap start

    > $sudo /etc/init.d/nfs-kernel-server start

    (8) 在客户端连接主机

    > $sudo mount 主机IP:/home/yourname/sharedir ~/nfsshare

    注意,nfsshare必须先存在。

    (9) 我们还可以设置允许的主机

    修改/etc/hosts.allow即可，其实不用修改，只要在/etc/exports中指明了主机就可以了
