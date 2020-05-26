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

#  实例方法同步
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

# 静态方法同步
使用`synchronized`关键字标记的静态方法就是静态方法同步块

```java
public static MyStaticCounter{

  private static int count = 0;

  public static synchronized void add(int value){
      count += value;
  }

}
```

同样，在这里`synchronized`关键字告诉Java `add（）`方法是同步的。

静态同步方法是同步在静态方法所属类的`class`对象上的。由于每个类在Java虚拟机中仅存在一个`class`对象，因此同事只允许一个线执行静态同步方法。

如果一个类包含多个静态同步方法，同实只能有一个线程可以在两个同步方法中的任何一个中执行。看下面的静态同步方法示例：

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

在任何给定时间，只有一个线程可以在两个`add（）`和`subtract（）`方法中的任何一个内执行。如果线程A正在执行`add（）`，那么直到线程A退出`add（）`为止，线程B都无法执行`add（）`或`subtract（）`。

如果静态同步方法位于不同的类中，一个线程可以执行每个类中的静态同步方法而无需等待。不管类中的那个静态同步方法被调用，一个类只能有一个线程调用其静态同步方法。

# 实例方法中同步块
有时你不需要同步整个方法，而只需同步方法中的一部分。Java可以对方法的一部分进行同步。

在非同步的Java方法中的同步代码块如下：

```java
  public void add(int value){

    synchronized(this){
       this.count += value;   
    }
  }
```

示例中使用Java同步构造器将一块代码标记为已同步。该代码执行时和同步方法效果一样。

注意，Java同步块构造器中传入了一个对象，在示例传的是`“ this”`，指的是当前`add`方法所在的那个对象实例。 同步构造器括号中传入的对象称为监视对象， 表示该代码是同步在该监视对象上。 同步实例方法将其所属的对象用作监视对象。

在同一监视对象上同步的Java代码块内只能执行一个线程。

以下两个示例都是同步在调用它们的实例对象上， 这样它们在同步方面是等效的：

```java
  public class MyClass {
  
    public synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

    public void log2(String msg1, String msg2){
       synchronized(this){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
  }
```

因此，在此示例中，只有一个线程可以在两个同步块中的任何一个中执行。

如果第二个同步块中的监视对象是不同的对象，则同一时间可以分别有两个不同的线程分别调用这两个方法。

# 静态方法中同步块

同步块也可以在静态方法中使用。 这是上一节中与静态方法相同的两个示例。 这些方法在方法所属的类的类对象上同步：

```java
 public class MyClass {

    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }

  
    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);  
       }
    }
  }
```

在同一时间，这两个方法中的任何一个都只能由一个线程执行。

如果第二个同步块是同步在与`MyClass.class`不同的对象上，则同一时间每个方法可以分别由一线程个执行。

# `Lambda`表达式中的同步块



