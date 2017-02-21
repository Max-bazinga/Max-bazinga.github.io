---
title: QEMU学习
date: 2017-02-17 11:37:27
tags: [QEMU]
---

# 什么是QEMU？
  QEMU是一个能够模拟处理器的自由软件。
  QEMU可以虚拟出不同架构的虚拟机，如在X86平台上可以虚拟出power机器。QEMU本身可以不以来于KVM，但是如果有KVM的存在并且硬件(处理器)支持比如Intel VT功能，那么QEMU在对处理器虚拟化着一块可以利用KVM提供的功能来提升性能。

# QEMU与KVM的却别？
  KVM是集成到Linux内核的Hypervisor，是X86架构且支持虚拟化技术的Linux的全虚拟化解决方案。从存在形式看，KVM是两个内核模块kvm.ko和kvm_intel.ko，这两个模块用来实现CPU的虚拟化。如果让用户在KVM上完成一个虚拟机相关的操作，显然需要用户空间的东西，同时还包括IO虚拟化，所以KVM的解决方案借鉴了QEMU的东西并做了一定的修改，形成了自己的KVM虚拟机工具集和IO虚拟化的支持，也就是所谓的qemu-kvm。
  QEMU是个独立的虚拟化解决方案，从这个角度它并不依赖KVM。而KVM是另一套虚拟化解决方案，不过因为这个方案实际上只实现了内核中对处理器虚拟化特性的支持，换言之，它缺乏设备虚拟化以及相应的用户空间管理虚拟化的工具，所以它借鉴了QEMU的代码并加以精简，连同KVM一起构成了另一个独立的虚拟化解决方案，称之为：qemu-kvm。

# 创建虚拟系统
  1. 创建硬盘镜像
     除非直接从CD-ROM 或者网络引导并且不安装系统到本地，运行QEMU时都需要硬盘镜像。硬盘镜像是一个文件，存储虚拟机硬盘上的内容。一个硬盘镜像可能是raw镜像，和客户机器上看到的内容一模一样，主机上占用的空间客户机上的大小一样。另一种方式是qcow2格式，仅当客户系统实际写入内容的时候，才会分配镜像空间。对客户机器来说，硬盘大小是完整大小，但是在主机系统上实际仅占用很小的空间。
     使用如下命令创建镜像：
     > qemu-img create /path/to/xp.img 10G

     创建虚拟机：
     > qemu-kvm -boot order=dc -vga qxl \
                     -spice port=3001,disable-ticketing -soundhw ac97 \
                     -device virtio-serial -chardev spicevmc,id=vdagent,debug=0,name=vdagent \
                     -device virtserialport,chardev=vdagent,name=com.redhat.spice.0 \
                     -cdrom /path/to/your.iso /path/to/your.img
     可能遇到的告警信息：
     > GLib-WARNING **: gmem.c:482: custom memory allocation vtable not supported
       可以忽略该告警信息，这是glib之前存在一个bug，最新版本已经修复了该bug。
