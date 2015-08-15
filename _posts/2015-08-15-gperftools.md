---
layout: post
title:  gperftools
date: 2015-08-15
---

用`c/c++`写过较大服务器程序的人，一般都应该知道有[valgrind](http://valgrind.org/)这个工具。一般是用来做内存泄露检查之类的工作，包括使用`massif`来追踪内存的使用情况。但是，用过的人也应该都记得，这个东西用起来真的是不爽，一方面是这个版本什么的有点问题，最重要的还是因为这个东西一旦跑起来，那程序是相当的慢。

不过不要紧，天下武功出Google，Google里面的各位大神应该是看不下去了，在写了`Tcmalloc`时，顺便就来了个`gperftools`。这里就不说伟大的`tcmalloc`性能比`Glibc malloc`高到哪里去了，主要说这个`gperftools`乃追bug神器啊。

整个`gperftools`包括了4方面内容。`tcmalloc`，`heap check`，`heap profiling`，`CPU profiler`这四个工具基本上可以解决`C/C++`程序中很多的bug了。

具体的使用方法，这里就不在多说了。主要是人家的文档写的太好了……

[文档地址](http://google-perftools.googlecode.com/svn/trunk/doc/index.html)