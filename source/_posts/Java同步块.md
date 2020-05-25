---
title: Java同步块
tags:
  - Java并发
categories:
  - Java并发
toc: false
---

Java中的同步块(`synchronized block`)用来标记方法或者代码块是同步的， 一次只能有一个线程执行同步块的内容，因此可以使用同步块来避免竞争。这篇文章详细地说明了同步关键字`synchronized`的工作方式。

# 并发工具集
`synchronized `是Java中第一个用于控制多个线程同步访问共享对象的机制。但是它并不足够优秀，所以在Java 5中又提供了一整套并发工具集（在`java.util.concurrent`包中），来帮助开发人员实现比同步块更细粒度的并发控制。

# `synchronized `关键字
Java中同步块是用``synchronized ``关键字标记的。同步块在Java中是同步在某个对象上的，所有同步在同一个对象上的同步块在同一时间只能有一个线程进入这些同步块中去执行操作，尝试进入这些同步块的所有其他线程将被阻塞，直到执行同步块的线程退出。

```synchronized ```关键字标记的同步块有以下四种类型：

1. 实例方法
2. 静态方法
3. 实例方法中的代码块
4. 静态方法中的代码块

这些同步块是同步在不同的对象上的，每一种类型将在下面更详细地说明。

#  实例方法同步块
这是一个同步的实例方法：

```java
public class MyCounter {

  private int count = 0;

  public synchronized void add(int value){
      this.count += value;
  }
}
```

注意在`add（）`方法声明中使用了`synchronized`关键字,这告诉Java该方法是同步的。

Java中的实例方法同步是同步在拥有该方法的实例（对象）上，因此，每个实例其同步方法都同步在不同的对象上，即该方法所属的实例。同一时间只能有一个线程能够在实例方法同步块中运行，如果存在多个实例，则每个实例都可以有个一个线程执行其实例方法同步块，即 一个实例一个线程。

对于同一对象（实例）中的所有同步实例方法都是一样的，因此，在下面的示例中，同一时间只有一个线程可以在两个同步方法中的任何一个中执行(一个实例一个线程)：



```java
public class MyCounter {

  private int count = 0;

  public synchronized void add(int value){
      this.count += value;
  }
  public synchronized void subtract(int value){
      this.count -= value;
  }

}
```

# 静态方法同步块
静态方法被标记为已同步，就像使用synced关键字的实例方法一样。这是一个Java同步静态方法示例：



```java
public static MyStaticCounter{

  private static int count = 0;

  public static synchronized void add(int value){
      count += value;
  }

}
```

同样在这里，synced关键字告诉Java add（）方法是同步的。

同步静态方法在同步静态方法所属类的类对象上同步。由于每个类的Java VM中仅存在一个类对象，因此同一类中的静态同步方法中只能执行一个线程。

如果一个类包含多个静态同步方法，则只能在一个方法中同时执行一个线程。看下面的静态同步方法示例：

```java
public static MyStaticCounter{

  private static int count = 0;

  public static synchronized void add(int value){
    count += value;
  }

  public static synchronized void subtract(int value){
    count -= value;
  }
}
```

在任何给定时间，只有一个线程可以在两个add（）和subtract（）方法中的任何一个内执行。如果线程A正在执行add（），那么直到线程A退出add（）为止，线程B都无法执行add（）或减去（）。

如果静态同步方法位于不同的类中，则一个线程可以在每个类的静态同步方法中执行。每个类一个线程，无论它调用哪种静态同步方法。

# 实例方法中的同步块
您不必同步整个方法。有时最好只同步一部分方法。方法内部的Java同步块使这成为可能。

这是未同步的Java方法中的Java代码的同步块：

```java
  public void add(int value){

    synchronized(this){
       this.count += value;   
    }
  }
```

本示例使用Java同步块构造将代码块标记为已同步。现在，将像执行同步方法一样执行此代码。

请注意，Java同步块构造如何在括号中使用对象。在示例“ this”中，使用了add方法。被带入的物体