---
layout: post
title:  where is ELF interpreter
date: 2014-11-02
---

最近遇到了一个编译、链接上的问题。之前我以为对这里已经很了解了，但是现在发现似乎还是有盲点的。这里还是记录下从这次问题中学习到的内容。

## 问题
在Linux下启动程序失败，失败日志如下

	xxx: bad ELF interpreter: No such file or directory`

出现这种问题其实也比较少见，如果不是因为我厂最近在升级系统，也不会有这个问题。为了避免以后将背景彻底遗忘，还是简单介绍下。为了说明方便A机器代表编译机，B机器代表执行机。

1. A、B机器系统默认的环境均是GCC3.4.3
2. 在A机器上使用GCC4.8.2.09编译可执行代码，动态连接
3. 在B机器上执行，B机器上对应GCC4.8.2只有GCC4.8.2.21

## 原因
与解释型语言不同，编译型语言需要经过编译后才能够执行。代码需要经过编译、链接才能够生成可执行代码，而可执行代码需要进行加载后才能够正常的执行。静态连接生成的可执行文件中包含了执行所需的所有内容，所以简单的进行加载就可以了。但是动态链接生成的代码在加载自身代码之前，需要将所依赖的动态库加载进来，而进行此工作的就是上述的`ELF interpreter`。
![alt text]({{ site.url }}/images/linker-loader.png)

问题就出现在在执行的机器上没有办法找到对应的`ELF interpreter`。

## 如何指定interpreter
那么，`ELF interpreter`这个东西是怎么被指定的呢？先来看看如何查看。

使用`readelf -l`命令可以查看可执行文件的每一段的header信息。对于动态连接的文件中，会有类似与`[Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]`内容，这里就是此二进制的`ELF interpreter`。

这个字段其实是在`.interp`段中，内容就是loader的地址。在编译时指定链接器`--dynamic-
linker`选项就可以指定这段的内容。

	[zhaochun ~]$gcc -Wl,--dynamic-linker=xxxx main.c
	[zhaochun ~]$readelf -l a.out
	
	Elf file type is EXEC (Executable file)
	Entry point 0x400430
	There are 8 program headers, starting at offset 64
	
	Program Headers:
	  Type           Offset             VirtAddr           PhysAddr
	                 FileSiz            MemSiz              Flags  Align
	  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
	                 0x00000000000001c0 0x00000000000001c0  R E    8
	  INTERP         0x0000000000000200 0x0000000000400200 0x0000000000400200
	                 0x0000000000000005 0x0000000000000005  R      1
	      [Requesting program interpreter: xxxx]

这里看到已经修改了`Requesting program interpreter`的内容。
此时再执行新生成的文件，就会复现最早遇到的问题了。

	[zhaochun ~]$./a.out
	-bash: ./a.out: xxxx: bad ELF interpreter: No such file or directory

## 解决问题的办法
可以有以下两种方式解决这个问题

1. 在编译可执行文件的时候使用`--dynamic-linker`选项来传递响应路径
2. 直接修改可执行ELF文件，[PatchELF](http://nixos.org/patchelf.html) 这个工具还是很给力的。

## 悬而未决的问题
虽然问题到了这，其实还是有些疑惑没有能够弄清楚。不知道以后能否有机会弄得清楚。

1. 什么时候完成`ELF interpreter`的调用
2. `ELF interpreter`具体又完成了什么样的工作
3. `SO`文件中的`ELF interpreter`字段什么时候用

以上这些问题还得在以后慢慢寻找答案了。