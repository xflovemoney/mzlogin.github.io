---
layout: post
title: Java多线程-从单例模式到Happens-Before
categories: Java,多线程
description: Java,多线程
keywords: Java,多线程
---

本文主要从简单的单例模式为切入点，分析单例模式可能存在的一些问题，以及如何借助Happens-Before分析、检验代码在多线程环境下的安全性。
### 知识准备

为了后面叙述方便，也为了读者理解文章的需要，先在这里解释一下牵涉到的知识点以及相关概念。
### 线程内表现为串行的语义
```
Within Thread As-If-Serial Semantics
```

### 定义

普通的变量仅仅会保证在该方法的执行过程中所有依赖赋值结果的地方都能获取到正确的结果，而不能保证变量赋值操作的顺序与程序代码中的执行顺序一致。
举个小栗子

### 看代码
```java
int a = 1;
int b = 2;
int c = a + b;
```
大家看完代码没准就猜到我想要说什么了。 假如没有重排序这个东西，CPU肯定会按照从上往下的执行顺序执行：先执行 a = 1、然后b = 2、最后c = a + b，这也符合我们的阅读习惯。 但是，上文也提及了：CPU为了提高运行效率，在执行时序上不会按照刚刚所说的时序执行，很有可能是b = 2 a = 1 c = a + b。对，因为只需要在变量c需要变量a``b的时候能够得到正确的值就行了，JVM允许这样的行为。 这种现象就是线程内表现为串行的语义。
### 重排序
#### 定义

指令重排序 为了提高运行效率，CPU允许讲多条指令不按照程序规定的顺序分开发送给各相应电路单元处理。 这里需要注意的是指令重排序并不是将指令任意的发送给电路单元，而是需要满足线程内表现为串行的语义
#### 现象

参照线程内表现为串行的语义一节中举的小栗子。

注意任何代码都有可能出现指令重排序的现象，与是否多线程条件下无关。在单线程内感受不到是因为单线程内会有线程内表现为串行的语义的限制。

### Happens-Before（先行发生）
**什么是Happens-Before**

**Happens-Before**原则是判断数据是否存在竞争、线程是否安全的主要依据。
```
为了叙述方便，如果操作X Happens-Before 操作Y，那么我们记为 hb(X,Y)。
```
如果存在hb(a,b)，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b。

* 是Java内存模型中定义的两项操作之间的偏序关系，满足偏序关系的各项性质 我们都知道偏序关系中有一条很重要的性质：传递性，所以Happens-Before也满足传递性。这个性质非常重要，通过这个性质可以推导出两个没有直接联系的操作之间存在Happens-Before关系，如： 如果存在hb(a,b)和hb(b,c)，那么我们可以推导出hb(a,c)，即操作a Happens-Before 操作c。

* 是判断数据是否存在竞争、线程是否安全的主要依据 这是《深入理解Java虚拟机》，375页的例子
```java    
i = 1;        //在线程A中执行
j = i;        //在线程B中执行
i = 2;        //在线程C中执行
```

假设线程A中的操作i = 1先行发生线程B的操作j = i，那么可以确定在线程B的操作执行后，变量j的值一定等于1，得出这个结论的依据有两个：一是根据先行发生原则，i = 1的结果可以被观察到；二是线程C还没有“登场“，线程A操作结束之后没有其他的线程会修改变量i的值。现在再来考虑线程C，我们依然保持线程A和线程B之间的先行发生关系，而线程C出现在线程A和线程B的操作之间，但是线程C与线程B没有先行发生关系，那j的值会是多少呢？答案是不确定！1和2都有可能，因为线程C对变量i的影响可能会被线程观察到，也可能不会，这时候线程B就存在读取到过期数据的风险，不具备多线程安全性。 通过这个例子我相信读者对Happens-Before已经有了一定的了解。

这里再重复一下Happens-Before的作用： 如果存在hb(a,b)，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b。

### Java 原生存在的Happens-Before

这些是Java 内存模型下存在的原生Happens-Before关系，无需借助任何同步器协助就已经存在，可以在编码中直接使用。



1. 程序次序规则（Program Order Rule） 在一个线程内，按照程序代码顺序，书写在前面的操作Happens-Before书写在后面的操作

* 管程锁定规则（Monitor Lock Rule） An unlock on a monitor happens-before every subsequent lock on that monitor. 一个unlock操作Happens-Before后面对同一个锁的lock操作。

* volatile变量规则（volatile Variable Rule） A write to a volatile field happens-before every subsequent read of that volatile. 对一个volatile变量的写入操作Happens-Before后面对这个变量的读操作。

* 线程启动规则（Thread Start Rule） Thread对象的start()方法Happens-Before此线程的每一个动作。

* 线程终止规则（Thread Termination Rule） 线程中的所有操作都Happens-Before对此线程的终止检测。

* 线程中断规则（Thread Interruption Rule） 对线程interrupt()方法的调用Happens-Before被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupt()方法检测到是否有中断发生。

* 对象终结规则（Finalizer Rule） 一个对象的初始化完成（构造函数执行结束）Happens-Before它的finalize()方法的开始。

* 传递性（Transitivity） 偏序关系的传递性：如果已知hb(a,b)和hb(b,c)，那么我们可以推导出hb(a,c)，即操作a Happens-Before 操作c。

这些规则都很好理解，在这里就不进行过多的解释了。 Java语言中无需任何同步手段保障就能成立的先行发生规则就只有上面这些了。

### 还存在其它的Happens-Before吗

Java中原生满足Happens-Before关系的规则就只有上述8条，但是我们还可以通过它们推导出其它的满足Happens-Before的操作，如：

* 将一个元素放入一个线程安全的队列的操作Happens-Before从队列中取出这个元素的操作
* 将一个元素放入一个线程安全容器的操作Happens-Before从容器中取出这个元素的操作
* 在CountDownLatch上的倒数操作Happens-Before CountDownLatch#await()操作
* 释放Semaphore许可的操作Happens-Before获得许可操作
* Future表示的任务的所有操作Happens-Before Future#get()操作
* 向Executor提交一个Runnable或Callable的操作Happens-Before任务开始执行操作

如果两个操作之间不存在上述的Happens-Before规则中的任意一条，并且也不能通过已有的Happens-Before关系推到出来，那么这两个操作之间就没有顺序性的保障，虚拟机可以对这两个操作进行重排序！

重要的事情说三遍：如果存在hb(a,b)，那么操作a在内存上面所做的操作（如赋值操作等）都对操作b可见，即操作a影响了操作b。

### volatile

初学者很容易将synchronized和volatile混淆，所以在这里有必要再两者的作用说明一下。 一谈起多线程编程我们往往会想到原子性、可见性，其实还有一个有序性常常被大家忘记。其实也不怪大家，因为只要能够保证原子性和可见性，就基本上能够保证有序性了，所以常常被大家忽略。

* 原子性 是指某个操作要么执行完要不不执行，不会出现执行到一半的情况。 synchronized和java.util.concurrent包中的锁都能够保证操作的原子性。
    * 可见性 即上一个操作所做的更改是否对下一个操作可见，注意：这里讨论的顺序是指时间上的顺序。
        * 一个被volatile修饰的变量能够保证任意一个操作所做的更改都能够对下一个操作可见

        * 上一条中讨论的原子操作都能对下一次相同的原子操作可见

        * 可以参照Happens-Before原则的第二、第三条规则
        
    
    * 有序性 Java中的有序性可以概括成一句话： 如果再本线程内观察，所有的操作都是有序的；如果再一个线程中观察另一个线程，所有的操作都是无序的。 前半句是指线程内表现为串行的语义（Within Thread As-If-Serial Semantics），后半句是指指令重排序现象和工作内存与主内存同步延迟现象。 首先volatile关键字本身就包含了禁止指令重排序的语义，而synchronized（及其它的锁）是通过“一个变量在同一时刻只允许一条线程对其进行lock操作”这条规则获得的，这条规则决定了持有同一个锁的两个同步块智能串行的进入。 注意：指令重排序在任何时候都有可能发生，与是否为多线程无关，之所以在单线程下感觉没有发生重排序，是因为线程内表现为串行的语义的存在。

    
#### volatile如何保证可见性
##### 可见性问题的由来

大家都知道CPU的处理速度非常快，快到内存都无法跟上CPU的速度而且差距非常大，而这个地方不加以处理通常会成为CPU效率的瓶颈，为了消除速度差带来的影响，CPU通常自带了缓存：一级、二级甚至三级缓存（我们可以在电脑描述信息上面看到）。JVM也是出于同样的道理给每个线程分配了工作内存（Woking Memory，注意：不是主内存）。我们要知道线程对变量的修改都会反映到工作内存中，然后JVM找一个合适的时刻将工作内存上的更改同步到主内存中。正是由于线程更改变量到工作内存同步到主内存中存在一个时间差，所以这里会造成数据一致性问题，这就是可见性问题的由来。

##### volatile采取的措施

volatile采取的措施其实很好理解：只要被volatile修饰的变量被更改就立即同步到主内存，同时其它线程的工作内存中变量的值失效，使用时必须从主内存中读取。 换句话说，线程的工作内存“不缓存”被volatile修饰的变量。

##### volatile如何禁止重排序

这个问题稍稍有点复杂，要结合汇编代码观察有无volatile时的区别。 下面结合《深入理解Java虚拟机》第370页的例子（本想自己生成汇编代码，无奈操作有点复杂）：

![](https://xflovemoney.github.io/images/blog/20161213113320688.jpg)


图中标红的lock指令是只有在被volatile修饰时才会出现，至于作用，书中是这样解释的：这个操作相当于一个内存屏障（Memory Barrier，重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；但如果有两个或者更多CPU访问同一块内存，且其中有一个在观测另一个，就需要内存屏障来保证一致性了。 重复一下：指令重排序在任何时候都有可能发生，与是否为多线程无关，之所以在单线程下感觉没有发生重排序，是因为线程内表现为串行的语义的存在。

### 分析双重检测锁（DCL）

哎，说了这么久终于到了双重检测锁（Double Check Lock，DCL）了，口水都说干了。大家是不是迫不及待的读下去了呢，嗯，我也迫不及待的写下去了。

这篇文章用happen-before规则重新审视DCL的作者在开头说到：
```
虽然99%的Java程序员都知道DCL不对，但是如果让他们回答一些问题，DCL为什么不对？有什么修正方法？这个修正方法是正确的吗？如果不正确，为什么不正确？对于此类问题，他们一脸茫然，或者回答也许吧，或者很自信但其实并没有抓住根本。
```
我觉得很对，记得一年前学习单例模式时，我也不懂为什么要加上volatile关键字，只是依葫芦画瓢跟着大家分析了一番，其实当时是不知道原因的。我相信有很多程序员也是我那时的心态。（偷笑

为了叙述方便，先把DCL的示例代码放在这里，后面分析时需要用到
```java
/**
 * Created by liumian on 2016/12/13.
 */
public class DCL {

    private static volatile DCL instance;

    private int status;

    private DCL(){
        status = 1;                         //1
    }

    private DCL getInstance(){
        if (instance == null){              //2
            synchronized (DCL.class){       //3
                if (instance == null){      //4
                    instance = new DCL();   //5
                }
            }
        }
        return instance;                    //6
    }

    public int getStatus(){
        return status;                      //7
    }
}
```
#### 在volatile的视角审视DCL

如果获取实例的方法使用synchronized修饰
```java
private synchronized DCL getInstance()
```
这样在多线程下肯定是没有问题的而且不需要加volatile修饰变量，但是会丧失部分性能，因为每次调用方法获取实例时JVM都需要执行monitorenter、monitorexit指令来进入和推出同步块，而我们真正需要同步的时刻只有一个：第一次创建实例，其余因为同步而花费的时间纯属浪费。所以缩小同步范围成为了提高性能的手段：只需要在创建实例时进行同步！于是将synchronized放入第一个if判断语句中并在同步代码块中在进行一次判空操作。那么问题来了： 假如没有volatile修饰变量会怎样？ 大家可能会说应该没啥问题啊，就是一行代码嘛：创建一个对象并把引用赋值给变量。没错，在我们看来就是一行代码，它的功能也很简单，但是，但是对于JVM来说可没那么简单了，至少有三个步骤（指令）：

1. 在堆中开辟一块内存（new）
* 然后调用对象的构造函数对内存进行初始化（invokespecial）
* 最后将引用赋值给变量（astore）

情形是不是跟上面重排序的例子很相似了呢？没错，假如没有volatile修饰，这些操作有可能发生重排序！JVM有可能这样做：

1. 先在堆中开辟一块内存（new）
* 马上将引用赋值给变量（astore）
* 最后才是调用对象的构造方法进行初始化（invokespecial）

好像在单线程下还是没问题，那我们把问题放在多线程情况下考虑（结合上面的DCL示例代码）： 假设有两条线程：T1、T2，当前时刻T1执行到语句1、T2执行到语句4，有可能会发生下面这个执行时序：

1. T2先执行，执行到语句5，但是此时JVM将三条指令进行了重排序：在时间上先执行new、astore、最后才是invokespecial
* 执行线程T2的CPU刚刚执行完new、astore指令，还没有来得及执行invokespecial指令就被切换出去了
* 线程T1现在登场了，执行if (instance == null)，因为线程T2已经执行了astore指令：将引用赋值给了变量，所以该判断语句有可能返回为false。如果返回为false，那么成功拿到对象引用。因为该引用所指向的内存地址还没有进行初始化（执行invokespecial指令），所以只要调用对象的任何方法，就会出错（会不会是NullPointerException？）

这就是不加volatile修饰为什么出错的一个过程。这时候有同学就会有疑问，按道理我不加volatile其它线程应该对我刚刚所做的修改（赋值操作）不可见才对呀。如果同学们这么想，我猜刚刚一定是把大家绕糊涂了：线程做的修改不应该对其它线程可见么？应该可见才对，理应可见。而volatile只是保证了可见性，就算没有它，可见性依然存在（不会保证一定可见）。

如果不了解volatile在DCL中的作用，很容易漏写volatile。这是我查资料时在百度百科上面发现的：

![](https://xflovemoney.github.io/images/blog/20161214104652147.png)


后面我给它加上去了：
![](https://xflovemoney.github.io/images/blog/20161214104759601.png)

### 利用Happens-Before分析DCL

经过前面的铺垫终于到了本片博客的第二个主题：利用Happens-Before分析DCL。

#### 先举个例子

在这篇文章中（happens-before俗解），作者提及到没有volatile修饰的DCL是不安全的，原因是（为了读者阅读方便，特将原文章的解释结合本文的代码）：语句1和语句7之间不存在Happens-Before的关系，大意是构造方法与普通方法之间不存在Happens-Before关系。为什么该篇文章作者提出这样的观点？我们来分析一下（注意此时没有volatile修饰）： 先抛出一个问题：语句7和哪些语句存在Happens-Before关系？ 我认为在线程T1中语句2与语句7存在Happens-Before关系，为什么？（这里只考虑发生线程安全问题的情况，如果执行到语句4了，就一定不会出现线程安全问题）请参照Happens-Before的第一条规则：程序次序规则（Program Order Rule），在一个线程内，按照程序代码顺序，书写在前面的操作Happens-Before 书写在后面的操作。准确的说，应该是控制流顺序而不是程序代码顺序，因为要考虑分支、循环等结构。 而语句2与语句7满足第一条规则，因为要执行语句7必须得语句2返回为false才能获取到对象的实例。然后语句2与语句6存在Happens-Before关系，原因同上。根据偏序关系的传递性，语句7与语句6存在Happens-Before关系，此外再也不能推出其它语句与语句7之间是否存在Happens-Before关系了，读者可以尝试推导一下。因为语句7与语句1，换句话说，普通方法与构造方法之间不存在Happens-Before关系，就算构造方法执行了，调用普通方法（如本例的getStatus()）也依然有可能得不到正确的返回值！JVM不保证构造方法所做的更改对普通方法（如本例的getStatus()）可见！

#### volatile对Happens-Before的影响

既然我们已经找到无volatile的DCL出现线程安全问题的原因了，解决起来就很轻松了，最简单的一个办法就是用volatile关键字修饰单例对象。（难道还有不使用volatile的解决办法？嗯，当然有，具体操作请留意后续博客）

现在我们来分析一下拥有volatile修饰的DCL带来了哪些不同？ 最显著的变化就是给变量（instance）带来了Happens-Before关系！请参考Happens-Before的第三条规则：volatile变量规则（Volatile Variable Rule），对一个volatile变量的写操作Happens-Before后面对这个变量的读操作，这里的“后面”指的是时间上的先后顺序。

有了volatile的加持，我们就可以推导出语句2 Happens-Before 语句5，只要执行了instance = new DCL();一定会被语句2instance == null观察到。读者此时可能又有疑问，上面就是因为语句5对语句2“可见”才出现问题的呀？怎么现在因为同样的原因反倒变成线程安全的了？别急，听我慢慢分析。嗯，刚刚的“可见”是打了双引号的，其实并不是整个语句5对语句2可见，而是语句5中的一条指令 - astore对语句2可见，并不包含invokespecial指令！因为volatile具有禁止重排序的语义，所以invokespecial一定在astore前面执行，换句话说构造方法一定在赋值语句之前执行，所以存在hb(语句1,语句5)，又因为hb(语句5,语句2)、hb(语句2,语句7)，所以推出hb(语句1,语句7) ——语句1 Happens-Before 语句7。现在将本例中的getStatus()方法和构造方法链接起来了，同理可以推出构造方法Happens-Before其它普通方法。


### 总结

本文分为两部分。
#### 第一部分

介绍了这几个知识点及相关概念：

* 线程内表现为串行的语义
* 重排序
* Happens-Before

#### 第二部分

通过两个角度（volatile、Happens-Before）对双重检测锁（DCL）进行了分析，分析为什么无volatile时会存在线程安全问题：

* volatile 因为指令重排序，而造成还没有构造完成就将对象发布了
* Happens-Before 因为普通方法与构造方法之间不存在Happens-Before关系

双重检测锁（DCL）所出现的安全问题的根本原因是对象没有正确（安全）的发布出去。 而解决这个问题的一种简单的方法就是使用volatile关键字修饰单例对象，从而解决线程安全问题。 读者可能会问，听你这么说，难道还有其它解决办法？我在上面也提到过，确实是还有其它方法，请留意后续博客，我将给大家带来不使用volatile关键字而保证线程安全的另一种方法。

