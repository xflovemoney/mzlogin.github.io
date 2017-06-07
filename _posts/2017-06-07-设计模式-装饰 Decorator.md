---
layout: post
title: 设计模式-装饰 Decorator
categories: 设计模式
description: 设计模式
keywords: 设计模式
---

### 如何给一个现有类扩展功能？
概述：
 若你从事过面向对象开发，实现给一个类或对象增加行为，使用继承机制，这是所有面向对象语言的一个基本特性。如果已经存在的一个类缺少某些方法，或者须要给方法添加更多的功能，你也许会仅仅继承这个类来产生一个新类—这建立在额外的代码上。  
 通过继承一个现有类可以使得子类在拥有自身方法的同时还拥有父类的方法。但是这种方法是静态的，用户不能控制增加行为的方式和时机。如果  你希望改变一个已经初始化的对象的行为，你怎么办？或者，你希望继承许多类的行为，怎么办?前一个，只能在于运行时完成，后者显然时可能的，但是可能会导致产生大量的不同的类—可怕的事情。

### 定义：
动态给一个对象添加一些额外的职责或者行为,就象在墙上刷油漆.使用Decorator模式相比用生成子类方式达到功能的扩充显得更为灵活。

### 意图：
当我们需要为某个现有的对象，动态的增加一个新的功能或职责时，可以考虑使用装饰模式。
当某个对象的职责经常发生变化或者经常需要动态的增加职责

### 类图
![](https://xflovemoney.github.io/images/blog/wqw212312122.png)
    
抽象组件角色(Component)：定义一个对象接口，以规范准备接受附加责任的对象，即可以给这些对象动态地添加职责。
具体组件角色(ConcreteComponent) :被装饰者，定义一个将要被装饰增加功能的类。可以给这个类的对象添加一些职责
抽象装饰器(Decorator):维持一个指向构件Component对象的实例，并定义一个与抽象组件角色Component接口一致的接口
具体装饰器角色（ConcreteDecorator):向组件添加职责。

#### 问题:  说装饰者模式比用继承会更富有弹性,在类图中不是一样用到了继承了吗?

说明:装饰者和被装饰者之间必须是一样的类型,也就是要有共同的超类。在这里应用继承并不是实现方法的复制,而是实现类型的匹配。因为装饰者和被装饰者是同一个类型,因此装饰者可以取代被装饰者,这样就使被装饰者拥有了装饰者独有的行为。根据装饰者模式的理念,我们可以在任何时候,实现新的装饰者增加新的行为。如果是用继承,每当需要增加新的行为时,就要修改原程序了。

#### 抽象装饰器
```java
package com.mystudy.decorator;

import com.mystudy.jdkproxy.IWork;

/**
 * Created by litf on 2017/6/2.
 */
public abstract class Decorator implements IWork {

    private  IWork iWork;

    public Decorator(IWork iWork){
        this.iWork = iWork;
    }
    public void WorkPro(){
        iWork.WorkPro();
    }
}
```
#### 具体装饰器角色
```java
package com.mystudy.decorator;

import com.mystudy.jdkproxy.IWork;

/**
 * Created by litf on 2017/6/2.
 */
public class DebuggerWork extends Decorator {
    public DebuggerWork(IWork iWork) {
        super(iWork);
    }

    public void Debugger() {
        System.out.println("今天调试了一天Bug");
    }

    @Override
    public void WorkPro() {
        Debugger();
        super.WorkPro();

    }
}
```

#### 具体装饰器角色
```java

package com.mystudy.decorator;

import com.mystudy.jdkproxy.IWork;

/**
 * Created by litf on 2017/6/2.
 */
public class CodeWork extends Decorator {
    public CodeWork(IWork iWork) {
        super(iWork);
    }

    public void CshapCode(){
        System.out.println("我开发的是C#。。。");
    }
    public void CodeLine(){
        System.out.println("我今天写了一百行代码");
    }

    @Override
    public void WorkPro(){
        CshapCode();
        CodeLine();
        super.WorkPro();

    }

}
```

#### 抽象组件角色
```java
package com.mystudy.jdkproxy;

/**
 * Created by litf on 2017/6/1.
 */
public interface IWork {

    void WorkPro();
}
```

#### 具体组件角色
```java
package com.mystudy.jdkproxy;

/**
 * Created by litf on 2017/6/1.
 */
public class JavaWork implements  IWork {

    public void WorkPro(){
        System.out.println("好好工作努力赚钱！！！");
    }
}
```

### 场景（理解装饰模式）
阿里的SimpleImage
#### Java IO的api
开始也提到了一点关于Java，io库的问题
java I/O库具有两个对称性，它们分别是：
  输入-输出对称：比如InputStream 和OutputStream 各自占据Byte流的输入和输出的两个平行的等级结构的根部；而Reader和Writer各自占据Char流的输入和输出的两个平行的等级结构的根部。
  byte-char对称：InputStream和Reader的子类分别负责byte和Char流的输入；OutputStream和Writer的子类分别负责byte和Char流的输出
这些作为根类，如果我们想通过缓冲，字节，或者是管道，这个时候我们就需要使用装饰器来进行装饰，然后通过装饰器来实现相应的操作，根类具有read方法，对于装饰类，通过构造函数将基类的一个实例注入进去，然后通过委托模式，首先通过基类的read方法获取字节流，然后根据相应的操作，实现字节读取等。

```java
InputStreamReader input = new InputStreamReader(System.in);
BufferedReader reader = new BufferedReader(input);
String line = reader.readLine();
```

### 优缺点
装饰者模式的设计原则为：对扩展开放、对修改关闭、依赖倒置原则
比静态继承更灵活
缺点：
有许多小对象 看起来复杂


