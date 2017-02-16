---
title: ssh监控的设计实现
date: 2017-02-15 16:57:54
tags: [ssh,advisor,design]
---

1. ##  调研目标

   旨在通过汇总集群中主机间的ssh连接信息，从而形成清晰的集群ssh连接托普图。

2. ## 实验环境准备
   一台物理kylin3.5系统，两台虚拟机kylin3.0系统。hostname分别修改为A(192.168.117.1), B(192.168.117.133), C（192.168.117.134）。
3. ## 实验步骤
   ### 实验一
   -  首先从A节点ssh到B节点，A->B,在B节点的/var/log/secure文件会生成两条日志记录：
   >  Feb  8 17:47:48 localhost sshd[12643]: Accepted password for root from 192.168.117.1 port 43171 ssh2
      Feb  8 17:47:48 localhost sshd[12643]: pam_unix(sshd:session): session opened for user root by (uid=0)

   -  然后再ssh到C节点，A->B->C，在C节点的/var/log/secure文件会生成以下两条日志记录：
   >  Feb  8 17:59:48 localhost sshd[9918]: Accepted password for root from 192.168.117.133 port 45169 ssh2
      Feb  8 17:59:48 localhost sshd[9918]: pam_unix(sshd:session): session opened for user root by (uid=0)

     如果同时在B上ssh到C，即存在A->B->C,B->C两条链路，那么根据现有的信息无法判断真实的连接到C上的情况是A->B->C,B->C还是A->B,B->C,B->C。在/var/run/utmp文件中保存有用户的登陆信息以及当前正在进行的操作，该文件是一个二进制文件，使用w命令能够解析获取到文件信息。例如存在一条A->B->C的链路，那么在B节点上通过w命令可以查看到如下信息：

   >  USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
   >  root     pts/0    192.168.117.1    09:34    1:51   0.02s  0.00s ssh root@192.168.117.134

   可以确定一个节点的两端连接节点。
   ### 实验二
   -  存在A，B，C，D四个节点，其中A节点上从w命令中获取到存在A->A, A->B两条链路；B节点上存在A->B->C,B->C两条链路；C节点上存在B->C,C->D,B->C->D三条链路；D节点上存在C->D,C->D两条链路。从而根据以上的信息，可以推断出该四个节点的ssh拓扑关系如下：
      ![](http://i1.piimg.com/567571/caa1b024c3929075.png)

<font color=red>其中B->C->D链路无法确定是A->B->C->D还是B->C->D。</font>
4. ## 算法实现
   - ###  在每台机器上跑一个agent，定时获取该机器w指令的结果上报到master服务器，存在以下两种格式的链路结果:
