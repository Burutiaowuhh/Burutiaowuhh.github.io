---
title: "equals 和 hashcode"
description: "equals 和 hashcode"
date: 2021-03-08T00:01:15+08:00
draft: false
tags: ["java"]
categories: ["java"]
featuredImagePreview: ""
---

Java 中的 Object 类是所有对象的公共父类。其中有两个方法。

- equals
- hashcode

所以每个对象都可以调用这两个方法。我们先来看看 equals 方法。

### 一、equals 方法

equals 方法通常是用来比较两个对象是否相等。这里我们要来跟`==`作一下区别。  

`==`在比较基本数据类型时是值比较。在比较引用数据类型的时候是比较对象的内存地址是否一样。  

  
而在 Object 类的 equals 方法中，对象之间是比较的内存地址是否相同。源代码如下：

```java
public boolean equals(Object obj) {
   return (this == obj);
}
```


  
可以看到，object 类中的 equals 比较的也就是对象在堆中的内存地址。  
[![6l6NUe.png](https://z3.ax1x.com/2021/03/08/6l6NUe.png)](https://imgse.com/i/6l6NUe)  
但是如果两个对象中的值相等我们就判断这两个对象相等，该怎么办呢？很简单，重写父类的 equals 方法。

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    A a = (A) o;
    return bName.equals(a.bName);
}
```

这是 idea 自动帮我们生成的 equals 方法。它不光帮我们判断了两个对象的内存地址，
还帮我们判断了对象是否为 null、对象的类加载器是否一致（从 jvm 层面来看，确定一个类的一致性，需要判断这个类的本身以及加载这个类的类加载器）还有对象中 bName 属性是否一致。


重写了 equals 方法后，我们再去判断两个对象是否相等。  
[![6l6tED.png](https://z3.ax1x.com/2021/03/08/6l6tED.png)](https://imgse.com/i/6l6tED)  
这次两个对象就是相等的了。

> 我们总结一下，对于`==`比较来说，基础类型是比较的值是否相同，对象比较的是内存地址是否相同。对于 equals 方法来说，如果不重写父类的 equals 方法，
> 那么默认就是使用 Object 类中的 equals 方法。如果自己重写了 equals 方法，那么就按照自己重写的 equals 方法来判断。

### 二、hashcode 方法

hashCode 方法也是 Object 中的方法。它的作用是获取这个对象的哈希码，这个哈希码的作用是确定该对象在哈希表（Hashmap,HashSet,HashTable）中的索引位置。虽然每个类都有 hashcode 方法，
但是只有当创建某个”类的散列表“时，该类的 hashCode 方法才有用，作用是确定该类在散列表中的位置，其他情况下，类的 hashCode 没有作用。

> 散列表的本质是通过数组实现的。当我们要获取散列表中的某个“值”时，实际上是要获取数组中的某个位置的元素。而数组的位置，就是通过“键”来获取的；更进一步说，数组的位置，是通过“键”对应的散列码计算得到的。

我们可以来看看 HashMap 中的源码。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这是 HashMap 中的 put 方法和 hash 方法。我们可以看到当我们调用 HashMap 中的 put 方法时，会对 key 值做 hash 计算，这个 hash 方法中就调用了我们今天的主角 hashCode 方法。计算出 hash 值后，再调用 putVal 方法去存键值对。


当我们把对象加入到 HashMap 中，HashMap 会先计算对象的 hashcode 值来判断对象加入的位置，同时也会与已经加入的对象的 hashcode 值作比较，如果没有相符的 hashcode，HashMap 会假设对象没有重复出现。但是如果发现有相同的 hashcode 值的对象,
这时会调用 equals 方法来检查 hashcode 相等的对象是否真的相同。如果不同的话就散列到其他位置，相同的话就覆盖原来的对象。

### 三、equals 和 hashCode 方法

面试的时候经常会被问到一道面试题：重写了 equals 方法后为什么要重写 hashCode 方法？  

首先要有一个概念，那就是：如果两个对象相等，那么它们通过 equals 方法比较一定相同，并且它们的 hashcode 值也是相同的；反过来，如果两个对象的 hashcode 值相同，不能说这两个对象就是相同的，还要再通过 equals 方法比较一下。那为什么两个对象的 hash 值相同不能确定这两个对象相同呢？  

因为 hash 计算存在一个哈希碰撞的问题，越好的哈希算法哈希碰撞的概率就越低，但是还是会存在哈希碰撞的情况。所以两个对象的哈希值相同，也有可能是这两个对象的 hash 计算值一样，但两个对象本身是不一样的。所以在散列表的对象比对中，
先通过 hashcode 值来比较两个对象，如果两个对象的 hash 相同，再去用 equals 方法比较。 
        
让我们来看一看，只重写 equals 方法是什么情况。  
  
对象 A 类重写了 equals 方法

```java
public class A {
    public String bName;

    public A(String bName) {
        this.bName = bName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        A a = (A) o;
        return bName.equals(a.bName);
    }
}
```


我们都知道，HashMap 是不允许存放重复 key 的。让我们看看测试类：  
[![6l6U4H.png](https://z3.ax1x.com/2021/03/08/6l6U4H.png)](https://imgse.com/i/6l6U4H)  
我们可以看到，虽然两个对象通过 equals 比较之后返回的是 true，也就说明两个对象实际上是相等的，但是当我们遍历了 HashMap 之后我们发现对象 a 和 b 都在 map 里面，并没有被覆盖掉。  
让我们重写 hashcode 方法。

  
对象 A 类重写了 equals 和 hashcode 方法

```java
public class A {
    public String bName;

    public A(String bName) {
        this.bName = bName;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        A a = (A) o;
        return bName.equals(a.bName);
    }

    @Override
    public int hashCode() {
        return Objects.hash(bName);
    }
}
```

测试类:  
[![6l6dCd.png](https://z3.ax1x.com/2021/03/08/6l6dCd.png)](https://imgse.com/i/6l6dCd)  
这里我们可以看到。重写了 hashcode 方法之后，map 中只能存在一个对象了。因为在 map.put(b)时，知道了 map 中已经存在相同的对象了，所以用新值覆盖掉旧值。
