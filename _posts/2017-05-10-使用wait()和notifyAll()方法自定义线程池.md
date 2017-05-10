---
layout: post
title: 使用wait()和notifyAll()方法自定义线程池
categories: Java
description: Java线程
keywords: Java线程
---

### 声明：

　　1、该篇只是提供一种自定义线程池的实现方式，可能性能、安全等方面需要优化；

　　2、该篇自定义线程池使用的是wait()和notifyAll()方法，也可以使用Lock结合Condition来实现；

　　3、该篇力求使用简单的方式呈现，如有错误之处，欢迎指正，在此表示感谢。

### 概述

　　自定义线程池三要素包括：

　　1、存储线程的容器（或叫线程池）。该容器可使用数组或链表，容器中存放执行线程，本篇使用链表实现。

　　2、执行线程（或叫执行器）。具体执行的线程。

　　3、执行任务。执行线程需要执行的具体任务。

### 代码

　　1、任务接口
```java
/**
 * 任务接口
 *
 */
public interface Task {
    
    /**
     * 执行具体任务
     */
    void executeSpecificTask();

}
```
2、执行线程接口
```java
/**
 * 执行线程接口
 *
 */
public interface Executor {
    
    /**
     * 设置执行任务
     * 
     * @param task 执行任务
     */
    void setTask(Task task);
    
    
    /**
     * 获取执行任务
     * 
     * @return Task
     */
    Task getTask();
    
    
    /**
     * 启动该任务
     */
    void startTask();

}
```
3、线程池接口　
```java
/**
 * 线程池接口
 *
 */
public interface Pool {
    
    /**
     * 获取执行线程
     * 
     * @return Executor
     */
    Executor getExecutor();
    
    /**
     * 销毁线程池容器中的所有执行线程
     */
    void destroy();
    
    
    /**
     * 获取线程池容器中执行线程数量
     * 
     * @return int
     */
    int getPoolSize();

}
```
4、自定义线程池具体实现
```java

/**
 * 自定义线程池具体实现
 *
 */
public class ThreadPool implements Pool{
    
    //是否关闭线程池，true-关闭线程池,false-不关闭线程池
    private boolean isShut;
    
    //默认执行线程个数
    private final static int DEFAULT_THREAD_SIZE = 6;
    
    //执行线程个数，默认为DEFAULT_THREAD_SIZE大小
    private static int THREAD_SIZE = DEFAULT_THREAD_SIZE;
    
    //存放执行线程的容器
    private LinkedList<Executor> list = new LinkedList<Executor>();
    
    public ThreadPool(){
        this(DEFAULT_THREAD_SIZE);
    }
    
    public ThreadPool(int threadSize){
        isShut = false;
        THREAD_SIZE = threadSize > 0 ? threadSize : DEFAULT_THREAD_SIZE;
        for(int i = 0;i < THREAD_SIZE;i++){
            Executor executor = new ExecutorImpl();
            ((ExecutorImpl)executor).setName("执行器" + i);
            
            //将执行器加入容器中
            list.add(executor);
            
            ((ExecutorImpl)executor).start();//启动执行器
        }
    }

    @Override
    public Executor getExecutor() {
        Executor executor = null;
        synchronized (list) {
            if(list.size() > 0){
                executor = list.removeFirst();
            }else{
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                executor = list.removeFirst();
            }
        }
        return executor;
    }

    @Override
    public void destroy() {
        synchronized (list) {
            isShut = true;
            list.notifyAll();
            list.clear();
        }
    }
    
    public  int getPoolSize(){
        synchronized (list) {
            return list.size();
        }
    }
    
    private class ExecutorImpl extends Thread implements Executor{
        
        //执行任务
        private Task task;
        
        //锁对象
        private final Object lock = new Object();
        
        public void run(){
            System.out.println(Thread.currentThread().getName() + " 启动>>>>>>>>>>");
            while(!isShut){
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + " 等待<<<<<<<<<<");
                    try {
                        //初始化线程池时，等待通知并释放锁
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println(Thread.currentThread().getName() + " 执行任务......");
                //执行具体任务
                getTask().executeSpecificTask();
                System.out.println(Thread.currentThread().getName() + " 执行任务完毕......");
                synchronized (list) {
                    System.out.println(Thread.currentThread().getName() + " 返回容器......");
                    //当前执行线程执行完毕后，返回到容器中
                    list.addFirst(ExecutorImpl.this);
                    //给别的执行线程发送通知
                    list.notifyAll();
                }
            }
        }
        
        public void startTask(){
            synchronized (lock) {
                lock.notify();
            }
        }

        @Override
        public void setTask(Task task) {
            this.task = task;
        }

        @Override
        public Task getTask() {
            return this.task;
        }
        
    }

}
```

5、执行任务实现　
```java
/**
 * 打印执行任务实现
 *
 */
public class PrintTask implements Task {

    @Override
    public void executeSpecificTask() {
        System.out.println("打印任务");
    }

}
```
6、测试
```java
public class Test {
    
    public static void main(String[] args) {
        
        Pool pool = new ThreadPool(2);
        
        for(int i = 0;i < 2;i++){
            Executor executor = pool.getExecutor();
            Task task = new PrintTask();
            executor.setTask(task);
            executor.startTask();
        }
        
        pool.destroy();
        
    }

}
```

7、测试结果
```java
执行器0 启动>>>>>>>>>>
执行器0 等待<<<<<<<<<<
执行器1 启动>>>>>>>>>>
执行器1 等待<<<<<<<<<<
执行器0 执行任务......
打印任务
执行器0 执行任务完毕......
执行器0 返回容器......
执行器1 执行任务......
打印任务
执行器1 执行任务完毕......
执行器1 返回容器......
```
