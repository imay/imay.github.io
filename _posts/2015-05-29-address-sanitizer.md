---
layout: post
title:  address sanitizer
date: 2015-05-29
---

这是很久之前的一个总结了，上次用了这个神器。这个礼拜又用了一次，真的是事半功倍啊。同时也发现了记忆不可靠，还是记下来更靠谱。这次发出来希望也能够帮助更多人。

## Address Sanitizer

在使用C++写代码的时候经常会遇到两类比较难追的问题，第一类就是内存泄露，第二类是访问（主要指写入）非法内存。对于第一部分，使用valgrind以及massif基本上能够比较顺利的解决问题。对于第二类解决方式，也可以使用valgrind来看，但是GCC4.8提供了非常强大的内存检测机制叫做“Address Sanitizer”[project](https://code.google.com/p/address-sanitizer/)。这个机制相对于valgrind的优势就是编译器集成，执行迅速，不像valgrind一样慢吞吞。

## 使用方法

这个工具使用方法也很简单。在编译的时候加入如下选项即可。

	CXXFLAGS="-fsanitize=address -fno-omit-frame-pointer "
	CFLAGS="-fsanitize=address -fno-omit-frame-pointer "
	LDFLAGS="-fsanitize=address"

简要的介绍以下各个参数的功能`-fsanitize=address`这个是开启内存越界访问的功能。`-fno-omit-frame-pointer`这个选项是为了让堆栈信息更加完善，便于追查。

如果程序使用上述编译选项，那么运行中一旦发生越界访问，就会自动退出，并且在标准错误输出(stderr)中输出对应越界访问的堆栈信息，借助一定工具就可以查看。

## 样例

测试代码如下：

	int main() {
	    char *a = new char[10];
	    for (int i = 0; i < 11; i++) {
	        a[i] = i;
	    }
	    return 0;
	}

使用上述选项进行编译，增加`-g`选项是为了方便查看堆栈信息。

	g++ -g -fsanitize=address -fno-omit-frame-pointer main.cc

运行程序

	./a.out 2>dump

查看dump信息，下述命令使用的脚本在[asan_symbolize.py](https://llvm.org/svn/llvm-project/compiler-rt/trunk/lib/asan/scripts/asan_symbolize.py)中

	cat dump | python asan_symbolize.py | c++filt

就会有如下信息

	==28113== ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60040000dffa at pc 0x400897 bp 0x7fff2af20f50 sp 0x7fff2af20f48
	WRITE of size 1 at 0x60040000dffa thread T0
	    #0 0x400896 (/home/disk1/zc/tmp/sanitizer/a.out+0x400896)
	    #1 0x7f57d9012bd4 (/home/opt/compiler/gcc-4.8.2.09/lib64/libc-2.18.so+0x21bd4)
	    #2 0x400718 (/home/disk1/zc/tmp/sanitizer/a.out+0x400718)
	0x60040000dffa is located 0 bytes to the right of 10-byte region [0x60040000dff0,0x60040000dffa)
	allocated by thread T0 here:
	    #0 0x7f57d9bcba3a (/home/opt/compiler/gcc-4.8.2.09/lib64/libasan.so.0.0.0+0x11a3a)
	    #1 0x400841 (/home/disk1/zc/tmp/sanitizer/a.out+0x400841)
	    #2 0x7f57d9012bd4 (/home/opt/compiler/gcc-4.8.2.09/lib64/libc-2.18.so+0x21bd4)
	Shadow bytes around the buggy address:
	  0x0c00ffff9ba0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9bb0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9bc0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9bd0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9be0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	=>0x0c00ffff9bf0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa 00[02]
	  0x0c00ffff9c00:fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9c10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9c20: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9c30: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
	  0x0c00ffff9c40: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa


已经足够分析出代码哪里发生了越界访问，而又是哪里申请的附近内存。

# 说明

1. 对于使用这个功能，**一定不能够**带有tcmalloc进行编译，因为tcmalloc是有线程内存Cache的，即使发生了非法访问，但是由于内存相当于是tcmalloc管理的，编译器并不会认为是越界访问，并不会有异常退出。所以要切记此点。

2. 为了保证便于追查问题，最好去掉编译选项中的优化选项。

# 总结

由于c++使用new或者malloc申请内存时候，越界访问也许当时不会Core，而是又运行了很久才可发生Core。而此时要找到问题发生的地方就比较困难了。所以有了这个利器还是很快的能够识别异常问题的。