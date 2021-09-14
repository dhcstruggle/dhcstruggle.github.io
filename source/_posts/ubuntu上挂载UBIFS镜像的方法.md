---
title: ubuntu上挂载UBIFS镜像的方法
date: 2021-07-29 10:48:44
tags:
- ubuntu
- ubifs
---

# 前言
最近手上有一个从芯片上直接通过RT809读取出来的镜像文件，需要拷贝出里面对应的文件，网络上查到的资料大多都不是很全，导致走了一些弯路，现把挂载方法记录于此。

<!--more-->

# 准备工作

## ubuntu虚拟机

	# uname -a
	Linux ubuntu 5.8.0-63-generic #71~20.04.1-Ubuntu SMP Thu Jul 15 17:46:08 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux


## ubifs镜像文件

	# file ubifs.bin
	ubifs.bin: UBI image, version 1
	# ls -l ubifs.bin
	-rw-r--r-- 1 root root 536870912 Aug  1 19:32 ubifs.bin


# 步骤

## 0x01
确认镜像的页大小和块大小：查找对应芯片的芯片手册，手册中有说明页大小和块大小。我手上的镜像页大小为4K，块大小256K


## 0x02
使用命令在dev目录下模拟出对应的mtd节点(需要足够大)，命令如下：

	# modprobe nandsim first_id_byte=0x98 second_id_byte=0xdc third_id_byte=0x90 fourth_id_byte=0x26
	# mtdinfo /dev/mtd0
	mtd0
	Name:                           NAND simulator partition 0
	Type:                           nand
	Eraseblock size:                262144 bytes, 256.0 KiB
	Amount of eraseblocks:          2048 (536870912 bytes, 512.0 MiB)
	Minimum input/output unit size: 4096 bytes
	Sub-page size:                  1024 bytes
	OOB size:                       128 bytes
	Character device major/minor:   90:0
	Bad blocks are allowed:         true
	Device is writable:             true

	# cat /proc/mtd
	dev:    size   erasesize  name
	mtd0: 20000000 00040000 "NAND simulator partition 0"


这里的四个id的参数，其中前两个代表对应产商ID和芯片ID，在以下的网站有列出各个参数代表的flash品牌和大小（取Full ID前四个参数即可）：
<http://www.linux-mtd.infradead.org/nand-data/nanddata.html>
另外，在网络上也有查到一些，这里可以参考
<http://trac.gateworks.com/wiki/linux/ubi#ubiondevhost>
<http://linux-mtd.infradead.org/faq/nand.html>
若模拟的mtd节点不符合要求，可输入“rmmod nandsim” 命令删除节点，重新模拟

## 0x03
把镜像文件导入到节点中：

	# dd if=ubifs.bin of=/dev/mtd0 bs=4096
	131072+0 records in
	131072+0 records out
	536870912 bytes (537 MB, 512 MiB) copied, 0.97402 s, 551 MB/s


或者：

	# nandwrite /dev/mtd0 ubifs.bin


## 0x04
把ubi文件系统和mtd0作关联

	# modprobe ubi mtd=/dev/mtd0,4096


## 0x05
挂载到对应的目录下即可

	# mount -t ubifs  -o ro /dev/ubi0_0 ubi


## 0x06
最后，如果要测试其它的ubifs镜像，需要移除以上步骤中加载的内核模块，移除的方法如下【前后顺序不可变】

	# rmmod ubifs ubi nandsim





