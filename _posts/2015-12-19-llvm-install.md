---
layout: post
title:  install llvm
date: 2015-12-19
---

# LLVM安装

本文记录的是LLVM3.3版本的源码编译方式

## 下载

下载对应的压缩包

[LLVM SRC](http://llvm.org/releases/3.3/llvm-3.3.src.tar.gz)

[Clang SRC](http://llvm.org/releases/3.3/cfe-3.3.src.tar.gz)

[Compiler RT](http://llvm.org/releases/3.3/compiler-rt-3.3.src.tar.gz)

下载完之后为
	
	[zc] ls
	cfe-3.3.src.tar.gz  compiler-rt-3.3.src.tar.gz  llvm-3.3.src.tar.gz

## 解压

解压下载的压缩包

	[zc] for i in `ls *.gz` ; do tar zxmf $i; done
	[zc] ls
	cfe-3.3.src  cfe-3.3.src.tar.gz  compiler-rt-3.3.src  compiler-rt-3.3.src.tar.gz  llvm-3.3.src  llvm-3.3.src.tar.gz
	[zc]

然后将`cfe`，以及`compiler-rt`移动到`llvm`目录中
	
	[zc] mv cfe-3.3.src llvm-3.3.src/tools/clang
	[zc] mv compiler-rt-3.3.src llvm-3.3.src/projects/
	
## 编译

创建`build`目录用于生成对应的中建文件

	[zc] mkdir build
	[zc] cd build/
	[zc] cmake -DLLVM_REQUIRES_RTTI:Bool=True -DLLVM_TARGETS_TO_BUILD="X86" -DCMAKE_BUILD_TYPE:STRING="RELEASE" -DLLVM_ENABLE_PIC=true -DCMAKE_INSTALL_PREFIX=`pwd`/../install ../llvm-3.3.src

这里需要说明的是，一定要指定`-DCMAKE_BUILD_TYPE:STRING="RELEASE"`这个选项，这样编译的时候会使用`-O3`选项，在动态生成代码的时候效率才会高。

然后可以 
	
	make && make install
























