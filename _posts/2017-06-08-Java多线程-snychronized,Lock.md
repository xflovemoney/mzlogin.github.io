---
layout: post
title: Java多线程-snychronized,Lock
categories: Java,多线程
description: Java,多线程
keywords: Java,多线程
---

* 当synchronized作用在方法上时，锁住的便是对象实例(this)。
* 当作用在静态方法时锁住的便是对象对应的Class实例。
* 因为Class数据存在于永久带，因此静态方法锁相当于该 类的一个全局锁
* 当synchronized作用于某一个对象实例时，锁住的便是 对应的代码块。

在HotSpot JVM实现中，synchronized锁有个专门的名字: 对象监视器。
synchronized是一个对象锁、悲观锁、非公平锁。

### JVM snychronized实现原理
 实现思路:基于链式队列实现。
 当多个线程同时请求某个对象监视器时，对象监视器会设置几种状态用来区分
请求的线程:
* ContentionList:所有请求锁的线程将被首先放置到该竞争队列
* EntryList:ContentionList中那些有资格成为候选人的线程被移到EntryList
* WaitSet:那些调用wait方法被阻塞的线程被放置到Wait Set
* OnDeck:任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck ü Owner:获得锁的线程称为Owner
* !Owner:释放锁的线程

新请求锁的线程将首先被加入到ConetentionList中，当某个拥有锁的线程 (Owner状态)调用unlock之后，如果发现 EntryList为空则从ContentionList 中移动线程到EntryList。
![](https://xflovemoney.github.io/images/blog/QQ20170608-150538@2x.png)


### 使用snychronized保证同步

* snychronized关键字总结:
snychronized关键字保证了原子性、可见性、有序性，实 现了线程安全。

* 缺点: 
1. 多线程竞争的情况下，频繁的加锁解锁导致过多的线程上下文切换，由于java线程是基于操作系统内核线程实现的， 所以如果阻塞或者唤醒线程都需要切换到内核态操作，这需
要耗费许多CPU资源。 
2. 一个线程持有锁，会导致其他请求该锁的线程挂起。 
3. 死锁

* 原子性 一个操作是原子操作，那么我们称它具有原子性。
原子操作(atomic operation):不可被中断的一个或一系列操作。
* 可见性 可见性指一个线程修改了共享变量的值，另外一个线程立即能够获得这个修改。 
* 有序性同一个线程内，所有操作都是有序的，从一个线程观察另外一个线程，都是无序的。
JVM会保证有序性。

### 使用Lock保证同步
![](https://xflovemoney.github.io/images/blog/QQ20170608-150810@2x.png)


#### 使用ReentrantLock保证同步
* 可重入锁ReentrantLock常用API  
```
public void lock()
```  
加锁，如果有别的线程获取了锁，则等待  
```
public void unlock()
```  
释放锁  
```
public final boolean isFair()
```  
判断该锁是否为公平锁  
```
public boolean tryLock()
```  
如果该锁未被别的线程占用，则加锁  
```
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException
```  
 在指定等待时间内，该锁未被别的线程占用，则加锁  
 
* 可重入锁ReentrantLock构造函数  
```
public ReentrantLock()
```  
构造可重入锁ReentrantLock对象，该锁为非公平锁  
```
public ReentrantLock(boolean fair)
```  
构造可重入锁ReentrantLock对象 如果fair为true,则该锁为公平锁 如果fair为false,则该锁为非公平锁  
ReentrantLock锁具有完成互斥排他的效果，即同一时间只有一个线程 在执行lock()后面的任务，通过unlock()释放锁，故保证了线程安全。  
ReentrantLock是一种乐观锁。  

* 可重入锁ReentrantLock使用  
```java
//可重入锁，非公平锁
private Lock lock = new ReentrantLock();
public void method(){ try{
lock.lock();//加锁
}finally{ lock.unlock();//释放锁
} }

//可重入锁，公平锁  
private Lock lock = new ReentrantLock(true);
public void waitMethod(){ try{
//在3秒内尝试获取锁
if(lock.tryLock(3, TimeUnit.SECONDS)){
System.out.println("在3秒内时间内，我获取锁了"); }else{
System.out.println("在3秒内时间内，我没有获取锁"); }
} catch (InterruptedException e) { e.printStackTrace();
}finally{
lock.unlock();//释放锁
} }
```

#### 使用ReentrantReadWriteLock保证同步

* 可重入读写锁ReentrantReadWriteLock
读写锁有两个锁，一个读操作相关的锁，称为读锁或共享锁，另一个写操作相关的锁，称为写锁或排他锁。
读锁之间不互斥，读锁与写锁互斥，写锁与写锁互斥。即多个线程可以同时进行读取操作，但同一时刻只允许一个线程进行写入操作。
* 构造函数
```
public ReentrantReadWriteLock()
```
 构造可重入读写锁ReentrantReadWriteLock对象，该锁是个非公平锁
```
public ReentrantReadWriteLock(boolean fair)
```
构造可重入读写锁ReentrantReadWriteLock对象 如果fair为true,则该锁为公平锁如果fair为false,则该锁为非公平锁

```java
可重入读写锁ReentrantReadWriteLock的使用
//可重入读写锁，非公平锁
private ReadWriteLock lock = new ReentrantReadWriteLock();
//读锁
private final Lock readLock = lock.readLock();
public void readMethod(){ try{
readLock.lock();//读锁 }finally{
  } }
  
可重入读写锁ReentrantReadWriteLock的使用
//可重入读写锁，公平锁
private ReadWriteLock lock = new ReentrantReadWriteLock(true);
//写锁
private final Lock writeLock = lock.writeLock();
public void writeMethod(){ try{
writeLock.lock();//写锁 }finally{
  } }
    
```
### Lock与synchronized区别
* Lock与synchronized都能保证线程安全，但Lock是一种乐观锁，而 synchronized是一种悲观锁。synchronized导致线程排队，而Lock 可以使线程在等待了足够长的时间以后，中断等待，而干别的事情。
* synchronized在代码执行时出现异常，JVM会自动释放锁，但是 Lock则不行，lock是通过代码实现的，要保证锁一定会被释放，就 必须将unLock()放到finally{}中。
* 在资源竞争不是很激烈的情况下，synchronized的性能要优于Lock， 但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十 倍，而Lock的性能能维持常态
注:JDK1.5中，对synchronized进行了优化，并且还有优化的空间， 因此我们写同步的时候，优先考虑synchronized。

### Volatile关键字
volatile关键字主要保证可见性。
* volatile关键字是线程安全的轻量级实现，所以其性能比 synchronized要好，并且volatile只能修饰于变量。
* 多线程访问volatile不会发生阻塞。
* volatile能保证数据的可见性，但不能保证原子性，故volatile并不能保证线程安全。 因此，volatile关键字一般用于状态判断中。
  
  