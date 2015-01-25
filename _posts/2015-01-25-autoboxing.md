---
layout: post
title:  autoboxing
date: 2015-01-25
---

## 问题

在写单测的时候遇到了一个奇葩问题，放到Map中的内容说什么也get不到，类似的代码如下

{% highlight java %}
public static void testMap() {
    Map<Long, String> map = new HashMap<Long, String>();

    map.put(100L, "hello");
    if (map.get(100) == null) {
        System.out.println("100 can not find in map");
    } else {
        System.out.println("value of 100 is hello");
    }
}
{% endhighlight java %}

当运行的时候输出的总是无法获得这个节点，这完全超出我之前的认识。但是隐约中感觉应该跟`get(100)`有关系，代码改成如下

{% highlight java %}
if (map.get(100L) == null) {
	...
}
{% endhighlight java %}

就能够按照我之前的意图走了。

## 自动装箱

为什么会出现这个问题呢？是因为Java中的[自动装箱(autoboxing)](http://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)技术导致。Java的编译器会自动将原始类型转化为对应的对象类型。对于上述我写的代码中

{% highlight java %}
if (map.get(100) == null) {
{% endhighlight java %}

此段代码，Java编译器会生成对应的代码

{% highlight java %}
if(map.get(Integer.valueOf(100)) == null) {
{% endhighlight java %}

编译器自动将`100`转化为了`Integer`对象。然后在Map中的`get`函数中会调用`Integer.equals(key)`来进行比较。而Map存储的key是`Long`类型，而`Integer`的`equals`函数如下

{% highlight java %}
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
{% endhighlight java %}

如果类型不相符是直接返回`false`的，所以，`get(100)`是肯定没有办法获取到`Long`对应的value的。

到这里问题原因基本就解决了，BUT，问题又来了，为什么`Map`的`get`方法参数是`Object`而不是对应的具体类型呢？

## Why object

对于为什么是object而不是对应具体的key类型，这种高level的事情，我自己是想不出来了，所以要感谢[Kevin Bourrillion](http://smallwig.blogspot.jp/2007/12/why-does-setcontains-take-object-not-e.html)这篇文章了。我这里简要的把他的思想翻译成中文。

好的，看下段代码

{% highlight java %}
public static void printNumberSet(Set<Number> values) {
    for (Number value : values) {
        System.out.println(value);
    }
}
{% endhighlight java %}

这段代码可以打印一个`Number`类型的`Set`，看上去很美，但是对于`Set<Long>`类型的参数，上面的函数就无能为力了。好吧，再看下段代码

{% highlight java %}
public static void printSet(Set<? extends Number> values) {
    for (Number value : values) {
        System.out.println(value);
    }
}
{% endhighlight java %}

这段代码可以解决上述的的问题，对于`Set<Long>`，`Set<Integer>`通通不惧啊。但是这里仍然存在一个问题，在`printSet`函数内部，是没有办法进行`values.add()`函数的，因为`Set.add()`需要具体的类型，而通过参数声明，编译器根本就没有办法确定其具体类型是什么。而`Set.contains()`这类的函数以`Object`为参数，那么仍然是可以使用的。所以这就是为什么对于`get`，`contains`这种函数要使用`Object`作为参数了。

好了，这里来说下这样设计API的总体思想。简要的就是：“对于写操作，严格要求；对于读操作，放松要求”。因为写会改变结果，而读并不会对整个数据结构造成毁灭性的破坏。当然还需注意`remove`属于写请求，但是使用`Object`做参数，可能是因为，他只能做减法吧。

## 后记

对于上述这种问题的避免，高大上的IDE是能够提供帮助的。譬如我使用的IntelliJ，会提供警告。如下

![alt text]({{ site.url }}/images/ide_warning.png)

所以，在写代码的过程中，还是有些洁癖比较好。
