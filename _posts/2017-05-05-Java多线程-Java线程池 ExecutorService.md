---
layout: post
title: Java多线程-Java线程池 ExecutorService
categories: Java,多线程
description: Java,多线程
keywords: Java,多线程
---

**** 本篇主要涉及到的是java.util.concurrent包中的ExecutorService。ExecutorService就是Java中对线程池的实现。

### 一、ExecutorService介绍

ExecutorService是Java中对线程池定义的一个接口，它java.util.concurrent包中，在这个接口中定义了和后台任务执行相关的方法：

![](https://xflovemoney.github.io/images/blog/20151027091338518.jpeg)

Java API对ExecutorService接口的实现有两个，所以这两个即是Java线程池具体实现类（详细了解这两个实现类，点击这里）：

1. ThreadPoolExecutor
2. ScheduledThreadPoolExecutor
除此之外，ExecutorService还继承了Executor接口（注意区分Executor接口和Executors工厂类），这个接口只有一个execute()方法，最后我们看一下整个继承树： 

![](https://xflovemoney.github.io/images/blog/20151027091552190.jpeg)

### 二、ExecutorService的创建

创建一个什么样的ExecutorService的实例（即线程池）需要g根据具体应用场景而定，不过Java给我们提供了一个Executors工厂类，它可以帮助我们很方便的创建各种类型ExecutorService线程池，Executors一共可以创建下面这四类线程池：

1. newCachedThreadPool 创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
2. newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
3. newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
4. newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

**** 备注：Executors只是一个工厂类，它所有的方法返回的都是ThreadPoolExecutor、ScheduledThreadPoolExecutor这两个类的实例。

### 三、ExecutorService的使用
```java
ExecutorService executorService = Executors.newFixedThreadPool(10);

executorService.execute(new Runnable() {
public void run() {
    System.out.println("Asynchronous task");
}
});

executorService.shutdown();
```
### 四、ExecutorService的执行

ExecutorService有如下几个执行方法：
```java
- execute(Runnable)
- submit(Runnable)
- submit(Callable)
- invokeAny(...)
- invokeAll(...)
```
#### 4.1 execute(Runnable)

这个方法接收一个Runnable实例，并且异步的执行，请看下面的实例：
```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

executorService.execute(new Runnable() {
public void run() {
    System.out.println("Asynchronous task");
}
});

executorService.shutdown();
```
这个方法有个问题，就是没有办法获知task的执行结果。如果我们想获得task的执行结果，我们可以传入一个Callable的实例（下面会介绍）。

#### 4.2 submit(Runnable)

submit(Runnable)和execute(Runnable)区别是前者可以返回一个Future对象，通过返回的Future对象，我们可以检查提交的任务是否执行完毕，请看下面执行的例子：
```java
Future future = executorService.submit(new Runnable() {
public void run() {
    System.out.println("Asynchronous task");
}
});

future.get();  //returns null if the task has finished correctly.
```
如果任务执行完成，future.get()方法会返回一个null。注意，future.get()方法会产生阻塞。

#### 4.3 submit(Callable)

submit(Callable)和submit(Runnable)类似，也会返回一个Future对象，但是除此之外，submit(Callable)接收的是一个Callable的实现，Callable接口中的call()方法有一个返回值，可以返回任务的执行结果，而Runnable接口中的run()方法是void的，没有返回值。请看下面实例：
```java
Future future = executorService.submit(new Callable(){
public Object call() throws Exception {
    System.out.println("Asynchronous Callable");
    return "Callable Result";
}
});

System.out.println("future.get() = " + future.get());
```

如果任务执行完成，future.get()方法会返回Callable任务的执行结果。注意，future.get()方法会产生阻塞。

#### 4.4 invokeAny(…)

invokeAny(...)方法接收的是一个Callable的集合，执行这个方法不会返回Future，但是会返回所有Callable任务中其中一个任务的执行结果。这个方法也无法保证返回的是哪个任务的执行结果，反正是其中的某一个。请看下面实例：
```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

Set<Callable<String>> callables = new HashSet<Callable<String>>();

callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 1";
}
});
callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 2";
}
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
    return "Task 3";
}
});

String result = executorService.invokeAny(callables);
System.out.println("result = " + result);
executorService.shutdown();
```
大家可以尝试执行上面代码，每次执行都会返回一个结果，并且返回的结果是变化的，可能会返回“Task2”也可是“Task1”或者其它。

#### 4.5 invokeAll(…)

invokeAll(...)与 invokeAny(...)类似也是接收一个Callable集合，但是前者执行之后会返回一个Future的List，其中对应着每个Callable任务执行后的Future对象。情况下面这个实例：
```java
ExecutorService executorService = Executors.newSingleThreadExecutor();

Set<Callable<String>> callables = new HashSet<Callable<String>>();

callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 1";
}
});
callables.add(new Callable<String>() {
    public String call() throws Exception {
    return "Task 2";
}
});
callables.add(new Callable<String>() {
public String call() throws Exception {
    return "Task 3";
}
});

List<Future<String>> futures = executorService.invokeAll(callables);

for(Future<String> future : futures){
System.out.println("future.get = " + future.get());
}

executorService.shutdown();
```

### 五、ExecutorService的关闭

当我们使用完成ExecutorService之后应该关闭它，否则它里面的线程会一直处于运行状态。

举个例子，如果的应用程序是通过main()方法启动的，在这个main()退出之后，如果应用程序中的ExecutorService没有关闭，这个应用将一直运行。之所以会出现这种情况，是因为ExecutorService中运行的线程会阻止JVM关闭。

如果要关闭ExecutorService中执行的线程，我们可以调用ExecutorService.shutdown()方法。在调用shutdown()方法之后，ExecutorService不会立即关闭，但是它不再接收新的任务，直到当前所有线程执行完成才会关闭，所有在shutdown()执行之前提交的任务都会被执行。

如果我们想立即关闭ExecutorService，我们可以调用ExecutorService.shutdownNow()方法。这个动作将跳过所有正在执行的任务和被提交还没有执行的任务。但是它并不对正在执行的任务做任何保证，有可能它们都会停止，也有可能执行完成。

[原文链接](http://blog.csdn.net/suifeng3051/article/details/49443835)