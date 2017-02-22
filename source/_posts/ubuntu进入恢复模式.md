---
title: ubuntu进入恢复模式
date: 2017-02-22 09:13:35
tags: [ubuntu,recovery]
---

1. 重起机器，然后迅速按shift键，进入grub引导界面
2. 选择advance模式，回车
3. ctrl+x进入系统
4. 选择root shell界面
5. 此时的文件系统是只读的，需要以读写的方式重新挂在根分区
   > mount -o remount,rw /

6. 使用下面的命令恢复之前修改的sudoers文件的权限
   > chmod 0440 /etc/sudoers
