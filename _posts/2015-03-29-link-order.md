---
layout: post
title:  link order
date: 2015-03-29
---

## 引子

使用`tcmalloc`有一段时间了。无需对代码进行修改，只用增加编译选项重新编译一次就能够将底层的分配器换成`tcmalloc`。这种无感的替换真的是贴心，本文其实不是想讲`tcmalloc`相关的内容，而是想讲一下链接相关的内容。

## 链接

此博主要的目的还是想记录下链接的顺序问题，这个问题之前自己也只是有个大概的印象，应该是什么样子的，也没有深入的了解。此博对于这个问题也没有深入的探讨，但是还是能够说明一些问题的，至少现阶段我遇到的问题还都能够解释的通。

为了后面讲解，先来两段代码吧

`main.c`的代码如下

{% highlight c %}
extern void my_func();
int main() {
    my_func();

    return 0;
}
{% endhighlight c %}

`func1.c`与`func2.c`的代码内容一致，内容如下：

{% highlight c %}
#include <stdio.h>

void my_func() {
    printf("%s %s:%d\n", __FILE__,  __FUNCTION__, __LINE__);
}
{% endhighlight c %}

## 实验

下面就可是来进行我的试验了。

### 实验一

`gcc main.c func1.c func2.c`

编译失败，原因是因为对于my_func有重复的实现。

### 实验二

`gcc -c func1.c -o func1.o`

`gcc -c func2.c -o func2.o`

`gcc main.c func1.o func2.o`

编译失败，错误同实验一，说明直接对对象文件的链接还是会产生同名的问题。

### 实验三

`gcc -c func1.c -o func1.o && ar rcs libfunc1.a func1.o`

`gcc -c func1.c -o func1.o && ar rcs libfunc2.a func2.o`

`gcc main.c libfunc1.a libfunc2.a`

编译成功。执行生成的二进制文件，发现调用的是`func1.c`中的`my_func`。这样很自然的就会想到是不是跟连接时传入的顺序有关系呢？

### 实验四

`gcc -c func1.c -o func1.o && ar rcs libfunc1.a func1.o`

`gcc -c func1.c -o func1.o && ar rcs libfunc2.a func2.o`

`gcc main.c libfunc2.a libfunc1.a`

前两步与实验三相同，只是最后一步改变了连接库的顺序。编译成功，执行生成文件，结果显示调用的是`func2.c`中的实现。

以上代码都在[这里](https://github.com/imay/snippet/tree/master/link_order)。

## 总结

经过上述的一些实验，总结一下吧：

1. 如果不同的对象文件中存在相同的实现，那么连接是不能成功执行的。
2. 如果不同的lib文件中有相同的实现，那么连接时会选择出现位置靠前的那一个。

回到主题说的`tcmalloc`，其实就是利用了第二条规则，内部实现了新的malloc，从而使得外部的函数能够先连接到`tcmalloc`的实现。

多说一句，为什么目标文件和库文件的行为不一样呢。其实也比较好理解，目标文件的含义是我都要包括，而库文件的含义则是按需索取。所以，这也是为什么库文件可以编译通过的原因了。