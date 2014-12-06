---
layout: post
title:  Java Unit Test
date: 2014-12-06
---

最近在用Java进行一些开发，在写的过程中需要进行单元测试工作。选用的是JUnit作为单测框架，后续使用了EasyMock，和PowerMock进行一些mock工作。EasyMock主要来完成对对象级别的Mock，而PowerMock则是用来完成对于静态方法、构造函数的Mock。

至于上述mock是怎么做的，暂时还没有进行深入研究。这篇博客主要是记录下怎么使用的，为的是后面再用的时候能够快速的使用。

## 要测试的类

有A，B，C三个类，其中A是我们要测试的类，A类中包含B类，又会调用C的静态函数。

{% highlight java %}
public class B {
    public boolean check() {
        return true;
    }
}

public class C {
    public static String hello() {
        return "Hello";
    }
}

public class A {
    private B b;

    public A(B b) {
        this.b = b;
    }

    public boolean check() {
        return b.check();
    }

    public void print() {
        C.hello();
    }
}
{% endhighlight %}

其中A是要进行测试的类，而B，C则是依赖的类，需要Mock掉

## EasyMock

使用EasyMock可以Mock掉依赖的B从而能够控制返回。具体的测试代码如下

{% highlight java %}
import junit.framework.Assert;
import org.easymock.EasyMock;
import org.junit.Test;

public class ATest {
	// 如果为了期待有异常抛出可以使用如下的方式
	// @Test(expected = XxxException.class)
    @Test
    public void testCheck() {
        B mockB = EasyMock.createMock(B.class);
        // 这里的anyTimes是为了保证check()多次调用的时候都会返回我们设定的值
        // 在anyTimes这里我就遇到了一个非常大的坑，
        // 结果搞了很久才发现是自己Mock的问题。
        EasyMock.expect(mockB.check()).andReturn(false).anyTimes();
        EasyMock.replay(mockB);
        // 将Mock的对象传入。
        A a = new A(mockB);
        Assert.assertFalse(a.check());
    }
}
{% endhighlight %}
	
上面就是EasyMock的一个基本的使用方式。稍微有点复杂的就是对于没有返回值的函数需要进行Mock的时候可以使用如下的方式。

正常的方式是

{% highlight java %}
EasyMock.expect(mockB.check()).andReturn(false).anyTimes();
{% endhighlight %}
	
如果对于没有返回值那么需要改成

{% highlight java %}
mockB.noReturnFunc();
EasyMock.expectLastCall().andXXX()
{% endhighlight %}
	
对于一般的情况，EasyMock就足够了。

## PowerMock

对于一些静态方法，以及New Object这种调用，那么EasyMock则是无能为力了。这时候就是PowerMock大显身手的时候了。对于上述对于C类静态方法调用的Mock方式如下

{% highlight java %}
import junit.framework.Assert;
import org.easymock.EasyMock;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.powermock.api.easymock.PowerMock;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;

// 这里需要指定使用PowerMockRunner，因为PowerMock可能需要修改一些字节码之类的东西
@RunWith(PowerMockRunner.class)
// 这里需要告诉PowerMock需要Mock什么类。多个类的话可以使用{A.class, B.class}方式
@PrepareForTest({C.class})
public class ATest {

    @Test
    public void testPrint() {
    	// 这里是说要Mock C类的静态方法。
        PowerMock.mockStatic(C.class);
        // 设定静态方法Mock掉的方式
        EasyMock.expect(C.hello()).andReturn("world").anyTimes();
        PowerMock.replay(C.class);

        A a = new A(new B());
        Assert.assertEquals("world", a.print());
    }
}
{% endhighlight %}

至此，两种Mock的方式就都介绍完了，基本上够用了。当然了，对于具体这些Mock的实现机制，我还没有去看，如果要真的明白了的话，可能会对Java会有更加深刻的了解。