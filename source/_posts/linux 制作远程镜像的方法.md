---
title: linux 制作远程镜像的方法
date: 2021-08-26 10:48:44
tags:
- linux
- dd
- nc
- ssh
---

# 前言
linux嵌入式开发过程中，有时候需要实时拷贝文件或者直接把emmc镜像导出，一般的方案是在设备端接入U盘，然后执行dd命令把镜像导入U盘中，或者是拷贝文件到U盘中，这种方式需要实际接触到设备，并且用U盘作为中间拷贝媒介才能把数据转移。有没有一种方法，在不接触设备的情况下，可远程制作设备镜像，或者远程拷贝文件？

<!--more-->

# 准备工作

## ubuntu虚拟机-测试机-开启sshd

	# uname -a
	Linux ubuntu 5.8.0-63-generic #71~20.04.1-Ubuntu SMP Thu Jul 15 17:46:08 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux


## windows 安装Mobaxterm - 主机

	 cat /proc/version
	 CYGWIN_NT-10.0-WOW version  (@) (gcc version 7.4.0 (GCC) ) Z

# 方法

## 0x01 使用ssh 配合 dd进行镜像制作
在windows端执行命令如下：

	ssh root@192.168.119.128 dd if=/dev/sda1 | dd of=/drives/c/Users/myuser/Desktop/sda1.bin

**此方法有一个限制，即如果ssh的账户没有root权限，则无法进行镜像制作。**

## 0x02 使用nc命令配合 dd进行镜像制作
在windows主机端运行：

	nc -l 6065 | dd of=/drives/c/Users/myuser/Desktop/sdc1.bin

这条命令的意思是：打开6065端口，把接收到的数据按字节写到sdc1.bin文件中
通过运行以下命令：

	netstat -a |grep 6065
	TCP    0.0.0.0:6065           DESKTOP-RCE4Q3T:0      LISTENING	

可以看到系统确实是开启了6065的端口。

在linux测试机端，运行如下命令：

	sudo dd if=/dev/sdc1 | nc 192.168.119.1 6065

这条命令的意思是：读取sdc1文件，数据发送到192.168.119.1的6065端口

综合起来意思就是：
把Linux端测试机的sdc1这个设备节点的数据复制到windows主机上，这样我们就通过网络实现了远程的镜像制作。
