---
title: ssh监控的设计实现
date: 2017-02-15 16:57:54
tags: [ssh,advisor,design]
---

1. ####  调研目标

   旨在通过汇总集群中主机间的ssh连接信息，从而形成清晰的集群ssh连接托普图。

2. ####  实验环境准备

   一台物理kylin3.5系统，两台虚拟机kylin3.0系统。hostname分别修改为A(192.168.117.1), B(192.168.117.133), C（192.168.117.134）。
3. #### 实验步骤
   ### 实验一

   *  首先从A节点ssh到B节点，A->B,在B节点的/var/log/secure文件会生成两条日志记录：

   >  Feb  8 17:47:48 localhost sshd[12643]: Accepted password for root from 192.168.117.1 port 43171 ssh2
      Feb  8 17:47:48 localhost sshd[12643]: pam_unix(sshd:session): session opened for user root by (uid=0)

   *  然后再ssh到C节点，A->B->C，在C节点的/var/log/secure文件会生成以下两条日志记录：

   >  Feb  8 17:59:48 localhost sshd[9918]: Accepted password for root from 192.168.117.133 port 45169 ssh2
      Feb  8 17:59:48 localhost sshd[9918]: pam_unix(sshd:session): session opened for user root by (uid=0)

      如果同时在B上ssh到C，即存在A->B->C,B->C两条链路，那么根据现有的信息无法判断真实的连接到C上的情况是A->B->C,B->C还是A->B,B->C,B->C。在/var/run/utmp文件中保存有用户的登陆信息以及当前正在进行的操作，该文件是一个二进制文件，使用w命令能够解析获取到文件信息。例如存在一条A->B->C的链路，那么在B节点上通过w命令可以查看到如下信息：

     >  USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
     >  root     pts/0    192.168.117.1    09:34    1:51   0.02s  0.00s ssh root@192.168.117.134

     因此，可以推断出存在一条A->B->C的链路，但是没有办法确定这条链路中各节点使用的tty号。

   ### 实验二

   -  存在A，B，C，D四个节点，其中A节点上从w命令中获取到存在A->A, A->B两条链路；B节点上存在A->B->C,B->C两条链路；C节点上存在B->C,C->D,B->C->D三条链路；D节点上存在C->D,C->D两条链路。从而根据以上的信息，可以推断出该四个节点的ssh拓扑关系如下：
      ![](http://i1.piimg.com/567571/caa1b024c3929075.png)

   <font color=red>其中B->C->D链路无法确定是A->B->C->D还是B->C->D。</font>
   **<font color=red>存在无法判断链路两端是建立在机器的哪个tty上的问题？</font>**
   经过对ssh协议的研究发现，在ssh连接建立过程的密钥交换阶段，服务器会向客户端发送一个包，这个包的内容包含自己的RSA主机密钥的公钥部分，服务密钥的公钥部分，支持的加密算法，支持的认证方法，次协议版本标志，以及一个64位的随即数(cookie)。客户端收到包后，根据这两把密钥和被称为cookie的64位随机数计算出会话session_id。服务器也是根据发给客户端的主机公钥，服务公钥，和64位随机数计算出会话ID，因此两边的会话ID是一致的。因此根据会话ID可以确立两边建立的一条会话链路。
   session_id的生成算法如下:
   ![](http://i1.piimg.com/567571/50d18170db2bb3ee.png)
   在sshd.c服务器程序下增加一条日志输出logit，可以打印出在服务器端生成的session_id。由于存在ssh1和ssh2两个协议，因此需要增加两条logit日志。
   ![](http://p1.bqimg.com/567571/03cce7be8909d3de.png)
   ![](http://p1.bqimg.com/567571/f73ab767c5149c43.png)
   确定了session_id后还需要确定与session_id相绑定的tty号，在服务器sshd的源代码上增加两条logit日志，输出连接及服务器端tty信息：
   ![](http://i1.piimg.com/567571/4d7074d67df4b3fe.png)
   在客户端增加两条logit，输出客户端session_id和tty信息：
   ![](http://i1.piimg.com/567571/4d7074d67df4b3fe.png)
   ![](http://p1.bpimg.com/567571/640c62a9c3ff500c.png)

   进行以下实验来验证*根据session_id来唯一确定一条链路的两端*：

   ### 实验三

   -  软件版本： openssh6.6, kylin3.0

     编译修改后的openssh源代码，生成新的sshd和ssh。使用下面的命令启动服务器程序，重定向日志到/tmp/sshd.log。
     > sshd需要以绝对路径启动：
       /home/zhanghao/openssh-6.6/openssh-6.6p1/sshd -E /tmp/sshd.log
     
     使用ssh连接服务器，其中client端输出如下日志信息：
     ![](http://i1.piimg.com/567571/20519a7f1ab75f2f.png)
     服务器端输出如下日志信息：
     ![](http://p1.bqimg.com/567571/a59f90843e597fa5.png)
     在对日志进行分析处理的过程中，比较两端的session_id信息，可以确定有客户端的tty2连接到了服务器的tty4。由于是在同一台机器上进行的连接，因此是从127.0.0.1到127.0.0.1。从而可以解决之前无法确定链路是A->B->C->D，还是B->C->D的问题。

