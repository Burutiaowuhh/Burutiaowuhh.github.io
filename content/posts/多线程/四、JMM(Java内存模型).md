---
title: "JMM（Java内存模型） （四）"
description: "JMM（Java内存模型）"
date: 2020-12-25T16:59:26+08:00
draft: false
tags: ["多线程","java"]
categories: ["多线程","java"]
featuredImagePreview: ""
---

### JMM 是什么

JMM 是一组规范，需要各个 JVM 的实现来遵守 JMM 规范，以便于开发者可以利用这些规范，更方便的开发多线程程序。  
JMM 是工具类和关键字的原理。volatile,synchronized,lock 的原理都是 JMM。

> 最重要的三个内容：重排序、可见性、原子性。

### 重排序

##### 什么是重排序

在线程 1 内部的两行代码的实际执行顺序和代码在 Java 文件中的顺序不一致，代码指令并不是严格按照代码语句顺序执行的，他们的顺序就被改变了，这就是重排序。让我们来看一段代码。

```java
public class OutOfOrderExection {

    private static int x = 0, y = 0;
    private static int a = 0, b = 0;

    public static void main(String[] args) throws InterruptedException {
        //用countdownlatch来设置栅栏 用于确保两个线程在同一时间启动
        CountDownLatch countDownLatch = new CountDownLatch(1);

        int i = 0;
        for (;;) {
            i++;
            x=0;
            y=0;
            a=0;
            b=0;

            Thread thread1 = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        countDownLatch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    a = 1;
                    x = b;
                }
            });
            Thread thread2 = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        countDownLatch.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    b = 2;
                    y = a;
                }
            });
            thread1.start();
            thread2.start();
            countDownLatch.countDown();
            thread1.join();
            thread2.join();

            String result = "第" + i + "次（" + x + "," + y + "）";
            System.out.println(result);
//            if (x == 0 && y == 0){
//                System.out.println(result);
//                break;
//            }else {
//                System.out.println(result);
//            }
        }
    }
}
```

此段代码运行后可能会出现四种情况。

1. 线程 1 运行完成后线程 2 再运行。此时 a 赋值为 1，x 赋值为 b 也就是 0，b 赋值为 2,y 赋值为 a 也就是 1，所以此时结果是 x=0,y=1。
2. 线程 2 先运行，然后线程 1 再运行。此时 b 赋值为 1，y 赋值为 a 也就是 0，a 赋值为 1，x 赋值为 b 也就是 2，所以此时结果是 x=2,y=0。
3. 情况三就是线程 1 和线程 2 交替运行。首先 a 赋值为 1，然后 b 赋值为 2，再接着 x 赋值为 b 也就是 2，y 赋值为 a 也就是 1，所以此时结果是 x=2,y=1。
4. 第四种情况是 y 赋值为 a 也就是 0，a 赋值为 1，x 赋值为 b 也就是 0，b 赋值为 2，所以此时结果就是 x=0,y=0。
   **第四种情况就是我们所要讨论的重排序现象。**

##### 重排序的好处

对比重排序前后的指令优化

[![ppH1sDx.png](https://s1.ax1x.com/2023/04/08/ppH1sDx.png)](https://imgse.com/i/ppH1sDx)

从图片中我们可以看到，经过重排序后的指令，节省了两条指令，提高了 cpu 的处理速度。

##### 重排序的 3 种情况

- 编译器优化：包括 JVM,JIT 编译器
- CPU 指令重排：就算编译器不发生重排，CPU 也肯对指令进行重排
- 内存的“重排序”：线程 A 的修改线程 B 看不到，引出可见性问题。

### 可见性

首先让我们来看一段代码

```java
public class FieldVisbility {

    int a = 1;
    int b = 2;

    private void change() {
        a = 3;
        b = a;
    }

    private void print() {
        System.out.println("b = " + b + ";" + "a = " + a);
    }

    public static void main(String[] args) {
        while (true){
            FieldVisbility fieldVisbility = new FieldVisbility();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    fieldVisbility.change();
                }
            }).start();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    fieldVisbility.print();
                }
            }).start();
        }

    }
}
```

这一段代码运行之后也是四种情况：

1. 线程 1 先运行完成后线程 2 运行，此时结果 a=3,b=3。

2. 线程 2 先运行完成后线程 1 开始运行，此时运行结果是 a=1，b=2。

3. 线程 1 和线程 2 交替运行，当线程 1 刚为 a 赋值为 3 的时候线程 2 开始执行打印方法，此时运行结果是 a=3，b=2。

4. **第四种情况就是遇见了可见性问题**，即线程 1 种将 a 赋值给 b，此时 a 还没有从线程 1 的本地内存种刷新到主内存种，线程 2 读取主内存中的 a 还是为 1，所以此时的结果就是 a=1,b=3。

简单图实例：

[![ppH1w8J.png](https://s1.ax1x.com/2023/04/08/ppH1w8J.png)](https://imgse.com/i/ppH1w8J)

###### 解决办法：对共享变量加上 volatile

```java
 volatile int a = 1;
 volatile int b = 2;
```

此时，如果线程 1 再更新 a 的内容，那么会强制将在线程 1 的本地内存中的 a 的值刷新到主内存之中。

##### 为什么会出现可见性问题

[![ppH1BvR.png](https://s1.ax1x.com/2023/04/08/ppH1BvR.png)](https://imgse.com/i/ppH1BvR)

从图中我们可以看到，在主存和核心之间存在着多级缓存，缓存之间的存储速度是不同的，速度最快的是 registers 寄存器。线程间的对于共享变量的可见性问题是由于缓存引起的而不是多核引起的。  
每个核心都会将自己需要的数据读到独占缓存中，数据修改后也是写入到缓存中，然后等待刷入到主存中。所以会导致有些核心读取的值是一个过期的值。

##### 什么是主内存和本地内存

[![ppH1yb6.png](https://s1.ax1x.com/2023/04/08/ppH1yb6.png)](https://imgse.com/i/ppH1yb6)

- 所有的变量都存储在主内存中，同时每个线程也有自己独立的工作内存，工作内存中的变量内容是主内存中的拷贝。
- 线程不能直接读写主内存中的变量，而是只能操作自己工作内存中的变量，然后再同步到主内存中。
- 主内存是多个线程共享的，但线程间不共享工作内存，如果线程间需要通信，必须借助主内存中转来完成。

> 所有的共享变量存在于主内存中，每个线程有自己的本地内存，而且线程读写共享数据也是统统佛本地内存交换的，所以才导致了可见性问题。

##### Happens-Before 原则

happens-before 规则是用来解决`可见性`问题：在时间上，动作 A 发生再动作 B 之前，B 保证能看见 A，这就是 Happens-Before。

- 单线程原则

- `锁操作（synchronized和Lock）`

  [![ppH1rK1.png](https://s1.ax1x.com/2023/04/08/ppH1rK1.png)](https://imgse.com/i/ppH1rK1)[![ppH1cVK.png](https://s1.ax1x.com/2023/04/08/ppH1cVK.png)](https://imgse.com/i/ppH1cVK)

- `volatile变量`

  由于 volatile 变量支持 Happens-Before 原则，所以在解决上述可见性问题时，只需要对第二个变量 b 添加 volatile 关键字即可。

  ```java
   int a = 1;
   volatile int b = 2;
  ```

  因为 volatile 关键字支持 Happens-Before 原则，所以在赋值方法中

  ```java
      private void change() {
          a = 3;
          b = a;
      }
      private void print() {
          System.out.println("b = " + b + ";" + "a = " + a);
      }
  ```

  在 print()方法中，对变量 b 进行读取操作时，一定会看到赋值变量 b 之前的全部操作，所以这里就可以看到 a 已经被赋值成 3。b 之前的写入（对应代码 b=a）对读取 b 后的代码（print b）都可见，所以在 writerThread 里对 a 的赋值，一定会对 readerThread 里的读取可见，所以这里的`a即使不加volatile，只要b读到是3,就可以由happens-before原则保证了读取到的都是3而不可能读取到1`。

  > 近朱者赤：给 b 加了 volatile，不仅 b 被影响，也可以`实现轻量级同步`

- 线程启动

- 线程 join

- 传递性

  如果 hb(A,B)而且 hb(B,C),那么可以推出 hb(A,C)

- 中断

  一个线程呗其他线程 interrupt 时，那么检测中断（isInterrupted）或者抛出 InterruptedException 一定能看到

- 构造方法

- `工具类`的 Happens-Before 原则
  - 线程安全的容器 get 一定能看到再次之前的 put 等存入动作
  - CountDownLatch
  - Semaphore
  - Future
  - 线程池
  - CyclicBarrier
----------------------------------------------------
笔记和图片来源：慕课网悟空老师视频[Java并发核心知识体系精讲](https://coding.imooc.com/class/362.html)