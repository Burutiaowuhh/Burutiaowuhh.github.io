---
title: "JVM 内存结构和 Java 对象模型"
description: "JVM 内存结构和 Java 对象模型"
date: 2020-12-13T20:07:15+08:00
draft: false
tags: ["java"]
categories: ["java"]
featuredImagePreview: ""
---

### JVM 内存结构

java 代码运行在虚拟机上的，虚拟机会把运行过程中的内存分为不同的区域，方柏霓管理，每一个区域有不同的作用。

[![ppH1f8H.png](https://s1.ax1x.com/2023/04/08/ppH1f8H.png)](https://imgse.com/i/ppH1f8H)

#### 堆

堆是整个运行区域中最大的一块。主要是 new 或其他指令创建的实例对象，
实例对象没有引用的会被垃圾回收，也包括 int 数组。堆的优势是在运行时动态分配。

#### 虚拟机栈

栈中保存了基本的数据类型以及对象的引用，对象的引用不是对象本身。特点是编译的时候就确定了大小，不会改变。

#### 方法区

方法区存储的主要是已经加载的各个 static 静态变量，类信息或者是常量信息。还包含永久引用，例如类前面修饰了 static，就存在方法区中。

#### 本地方法栈

主要存 native 方法

#### 程序计数器

保存当前线程所执行的字节么的行号数，在上下文切换时会保存下来。还包括下一条执行的指令，分支和循环等异常处理。

> 方法区、堆（所有线程共享）Java 栈、本地方法栈、程序计数器（线程私有）

### Java 对象模型

[![ppH1h2d.png](https://s1.ax1x.com/2023/04/08/ppH1h2d.png)](https://imgse.com/i/ppH1h2d)

[![ppH14xA.png](https://s1.ax1x.com/2023/04/08/ppH14xA.png)](https://imgse.com/i/ppH14xA)

1. JVM 会给这个类创建 instanceKlass,保存在方法区，用来在 JVM 层表示该 Java 类
2. 当我们在 Java 代码中使用 new 创建一个对象的时候，JVM 创建一个 instanceOopDesc 对象，这个对象中包含了对象头和实例数据。
3. java 代码对实例对象进行调用时，会将对象引用存在栈中

---

笔记和图片来源：慕课网悟空老师视频[Java 并发核心知识体系精讲](https://coding.imooc.com/class/362.html)
