---
layout: post
title:  __builtin_unreachable
date: 2015-07-15
---

距离上次更新博客都已经有45天了。这期间有自身周期性迷失的问题，也有意外的问题。总之是博客断了有一阵了，今天借着这个简单的东西恢复起来吧。

## __builtin_unreachable

这个是`GCC`的一个内置的函数，具体的内容可以参考[这里](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)。主要是用于终止当前函数的执行。直接上例子吧

	#include <stdio.h>
	
	int foo(int argc) {
	    if (argc != 1) {
	        return argc;
	    }
	    __builtin_unreachable();
	    printf("Reach Here\n");
	}
	
	int main(int argc, char* argv[]) {
	    printf("Hello world %d\n", foo(argc));
	    return 0;
	}
	
对于上述的代码，使用`gcc -Wall `编译是不会提示有问题的。但是如果注释掉`__builtin_unreachable();`这一行，那么会有`warning: control reaches end of non-void function [-Wreturn-type]`这样的警告的。

所以这个一般都是用于告诉编译器，走到这里都是错啦，就不要在关注是不是有返回值啦。譬如给的例子如下

	int f (int c, int v)
    {
      if (c) {
          return v;
      } else {
          asm("jmp error_handler");
          __builtin_unreachable ();
      }
    }

好啦，这个东西是自己又学到的内容，虽然知识点很少，但是还是比较有意思的。




