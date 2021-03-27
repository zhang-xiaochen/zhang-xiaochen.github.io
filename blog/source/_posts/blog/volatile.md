---
title: volatile关键字使用指南
tags: [volatile]
categories: [并发编程]
date: 2021-03-27 15:30:00
---
# volatile关键字使用指南

```java
package com.example.way.thread;


public class TaskRunner {
    private int number;
    public void add(int number){
        this.number = number;
    }

    public int getNumber(){
        return number;
    }


    public static void main(String[] args) throws InterruptedException {
        TaskRunner taskRunner = new TaskRunner();
        new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            taskRunner.add(100);
            System.out.println(Thread.currentThread().getName() + " over");
        }).start();

        while (true){
            if (taskRunner.getNumber() == 100) {
                System.out.println(Thread.currentThread().getName() + " over");
                return;
            }
        }
    }
}

```

​        我们期望的运行结果是新线程修改完number的值以后，主线程能够获取到新的值，从而结束线程，但结果并非如此：

![image-20210327153904240](/img/volatile/image-20210327153904240.png)

将number字段改为volatile修饰以后：

![image-20210327161321414](/img/volatile/image-20210327161321414.png)

## 内存可见性

>  可见性指一个线程对共享变量的修改能够及时的被其他线程看到。

​        JVM中所有变量都存储在主内存中，每个线程都有自己独立的本地内存，用来保存使用到的变量副本，并且，线程对于共享变量的操作都必须再自己的工作内存中进行，不能直接从主内存中读写，不同线程之间无法直接访问其他线程工作内存中的变量，线程之间的变量值传递需要通过主内存来完成。

​        Java 提供了一种稍弱的同步机制，即 volatile 变量，用来`确保将变量的更新操作通知到其它线程`。当把共享变量声明为 volatile 类型后，线程对该变量修改时会将该变量的值`立即刷新回主内存`，同时会使其它线程中`缓存的该变量无效`，从而其它线程在读取该值时会从主内中重新读取该值（参考缓存一致性）。因此在读取 volatile 类型的变量时总是会返回最新写入的值。

![image-20210327161709277](/img/volatile/image-20210327161709277.png)

## 指令重排

​        为了提高性能，编译器和处理器会对代码编译后的指令进行重新排序。

* 编译器会在不改变单线程语义的情况下，对语句进行重排。
* 如果代码中某些语句之间不存在数据依赖，处理器可以对机器指令进行排序。
* 高速缓存的存在使程序的读写顺序可能是错乱的。

![image-20210327161709277](/img/volatile/cpu.png)

`在JDK1.5之后，可以使用volatile变量禁止指令重排序。`

## volatile 和 synchronization

​        多线程应用中，我们必须确保互斥性和可见性。

* synchronization修饰的方法和代码块可以实现互斥性和可见性。
* volatile关键字只能确保可见性。

​        volatile相较于synchronization更轻量，不会阻塞线程。在不需要互斥的情况下比较有用。

## Happens-Before原则

JVM定义的Happens-Before原则是一组偏序关系：`对于两个操作A和B，这两个操作可以在不同的线程中执行。如果A Happens-Before B，那么可以保证，当A操作执行完后，A操作的执行结果对B操作是可见的。`

Happens-Before的规则包括：

1. 程序顺序规则
2. 锁定规则
3. volatile变量规则
4. 线程启动规则
5. 线程结束规则
6. 中断规则
7. 终结器规则
8. 传递性规则

```java

public class TaskRunner {
    private int number;
    private volatile boolean ready;
    public void setNumber(int number){
        this.number = number;
    }
    public boolean getReady(){
       return this.ready;
    }

    public void setReady(boolean ready){
        this.ready = ready;
    }
    public int getNumber(){
        return number;
    }


    public static void main(String[] args) throws InterruptedException {
        TaskRunner taskRunner = new TaskRunner();
        new Thread(() -> {
            taskRunner.setNumber(100);
            taskRunner.setReady(true); // ready使用volatile修饰 根据Happend-Before原则 该线程的操作结果是可以被下面的线程看到的
        }).start();
        new Thread(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if(taskRunner.getReady()){
                System.out.println(taskRunner.getNumber());
            }
        }).start();
    }
}
```

执行结果：

![image-20210327173906892](/img/volatile/image-20210327173906892.png)



