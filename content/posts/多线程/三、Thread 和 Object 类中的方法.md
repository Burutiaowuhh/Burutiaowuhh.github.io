---
title: "Thread 和 Object 类中的方法 （三）"
description: "Thread 和 Object 类中的方法"
date: 2020-12-20T11:21:26+08:00
draft: false
tags: ["多线程","java"]
categories: ["多线程","java"]
featuredImagePreview: ""
---

### Object 类

- wait
- notify/notifyAll

### Thread 类

- sleep
- join
- yield
- currentThread
- start/run
- interrupt
- stop/suspend/resume

线程调用 wait 方法进入阻塞状态，直到以下四种情况会被唤醒：

- 另一个线程调用 notify 方法
- 另一个线程调用 notifyAll 方法
- 过了 wait(long timeout)规定的超时时间，如果传入 0 就是永久等待
- 线程自身调用了 interrupt

> wait 会释放 monitor 锁。只释放当前 monitor

##### wait,notify,notifyAll 特点、性质

1. 必须先拥有 monitor  
2. 属于 Object 类  
3. 类似功能 condition

[![rOKU9x.png](https://z3.ax1x.com/2020/12/30/rOKU9x.png)](https://imgse.com/i/rOKU9x)

##### 用 wait、notify 实现生产者消费者模式

[![rOKH5n.png](https://z3.ax1x.com/2020/12/30/rOKH5n.png)](https://imgse.com/i/rOKH5n)

[![rOKqCq.png](https://z3.ax1x.com/2020/12/30/rOKqCq.png)](https://imgse.com/i/rOKqCq)

仓库

```java
class EventStorage{

    int max = 10;
    LinkedList<Date> list;

    public EventStorage() {
        this.max = 10;
        this.list = new LinkedList<>();
    }

    public synchronized void put(){
        while (list.size() == max){
            System.out.println("仓库已满");
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        list.add(new Date());
        System.out.println("当前添加的是第"+list.size()+"个元素");
        notify();
    }

    public synchronized void take(){
        while (list.size() == 0){
            System.out.println("仓库已空");
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        list.poll();
        System.out.println("当前仓库还剩余"+list.size()+"个元素");
        notify();
    }

}
```

生产者

```java
class Producer implements Runnable{

    private EventStorage eventStorage;

    public Producer(EventStorage eventStorage) {
        this.eventStorage = eventStorage;
    }

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            eventStorage.put();
        }
    }
}
```

消费者

```java
class Consumer implements Runnable{
    private EventStorage eventStorage;

    public Consumer(EventStorage eventStorage) {
        this.eventStorage = eventStorage;
    }

    @Override
    public void run() {
        for (int i = 0;i < 100;i++) {
            eventStorage.take();
        }
    }
}
```

调用

```java
public class ProducerConsumerModel {

    public static void main(String[] args) {
        EventStorage eventStorage = new EventStorage();
        Producer producer = new Producer(eventStorage);
        Consumer consumer = new Consumer(eventStorage);
        Thread thread1 = new Thread(producer);
        Thread thread2 = new Thread(consumer);
        thread1.start();
        thread2.start();

    }
}
```

> 注意：使用 wait 方法须在同步代码块中

###### 用两个线程交替打印 0~100 的奇数偶数

方法一、使用 synchronized 关键字实现

```java
public class WaitNotifyPrintOddEvenSyn {

    private static int count;
    private static Object lock=new Object();
    //新建2个线程
    //1个打印奇数，一个打印偶数（用位运算）
    //用synchronized来通信
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                while (count<100){
                    synchronized (lock){
                        if ((count & 1) == 0){
                            System.out.println(Thread.currentThread().getName()+":"+count++);
                        }
                    }
                }
            }
        },"偶数").start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                while (count<100){
                    synchronized (lock){
                        if ((count & 1) == 1){
                            System.out.println(Thread.currentThread().getName()+":"+count++);
                        }
                    }
                }
            }
        },"奇数").start();


    }
}
```

两个线程之间互相抢锁，抢到锁的线程去运行。

> 问题：有许多操作是废操作，当线程 2 没有抢到锁时，线程 1 一直在进行无用操作，浪费了很多资源

方法二、使用 wait、notify 实现

```java
public class WaitNotifyPrintOddEveWait {
    private static int count = 0;
    private static Object lock=new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new TurningRunner(),"偶数");
        thread1.start();
        Thread.sleep(100);
        Thread thread2 = new Thread(new TurningRunner(),"奇数");
        thread2.start();
    }

    //1.拿到锁，我们就打印
    //2.打印完，唤醒其他线程，自己就休眠
    static class TurningRunner implements Runnable{
        @Override
        public void run() {
            while (count<100){
                synchronized (lock){
                    System.out.println(Thread.currentThread().getName()+":"+count++);
                    lock.notify();
                    if (count<100){
                        try {
                            lock.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
```

> 比 synchronized 节省了很多不必要的操作，节省了资源

为什么线程通信的方法 wait(),notify(),notifyAll 被定义在 Object 类里？而 sleep 定义在 Thread 类里？

> wait().notify().notifyAll()是锁级别的操作，而锁属于每一个对象的，对象头中用来标记锁状态，锁是绑定在每一个对象中。

> Thread 类会进行 notify 操作

##### sleep 方法详解

作用： 我只想让线程在预期的时间执行，其他时间不要占用 CPU 资源

sleep 方法不释放锁

- 包括 synchronized 和 lock
- 和 wait 不同，sleep 不释放锁

###### wait/notify、sleep 异同

- 相同
  - 阻塞
  - 响应中断
- 不同
  - 同步方法中
  - 释放锁
  - 指定时间
  - 所属类


-------------------------------------------------

笔记和图片来源：慕课网悟空老师视频[Java并发核心知识体系精讲](https://coding.imooc.com/class/362.html)