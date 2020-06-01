---

title: Java Volatile 关键字
tags:
  - Java并发
categories:
  - Java并发
toc: false
---

Java中 **`volatile`**关键字用于将Java变量标记为“**将存储在主存储器中**”。更准确地说，这意味着将对**`volatile`**变量的每次读取都将从计算机的主内存中读取，而不是从CPU缓存中读取，并且对**`volatile`**变量的每次写入都将写入主存储器中，而不仅是CPU高速缓存中。

实际上，从Java 5开始，**`volatile`**关键字不仅仅保证将**`volatile`**变量写入主存储器或从主存储器读取。我将在以下各节中进行解释。

# 变量的可见性问题
` volatile`关键字可确保跨线程更改变量的可见性。这听起来有点抽象，所以让我详细说明。

在多线程应用程序中，线程对`non-volatile`变量进行操作，出于性能方面的考虑，每个线程在对其进行操作时都可以将变量从主内存复制到CPU缓存中。如果您的计算机包含多个CPU，则每个线程可能在不同的CPU上运行。这意味着，每个线程可以将变量复制到不同CPU的CPU缓存中。这在这里说明：

![Threads may hold copies of variables from main memory in CPU caches.](http://img.zhoubg.cn/static/20200530153630.png)

对于`non-volatile`变量，无法保证Java虚拟机何时将数据从主存储器读取到CPU缓存中，或何时将数据从CPU缓存写入到主存储器中。 这可能会导致一些问题，我将在以下各节中进行解释。

设想一种情况，两个或多个线程可以访问一个共享对象，该共享对象中声明了一个计数器变量`counter`：

```java
public class SharedObject {

    public int counter = 0;

}
```

想象一下，只有线程1会递增`counter`变量，但是线程1和线程2都可能会不时的读取`counter`变量。

如果`counter`变量未声明为volatile，则无法保证何时将`counter`变量的值从CPU高速缓存写回主存储器。 这意味着，CPU缓存中的`counter`变量值可能与主存储器中的不同。 此处说明了这种情况：

![](http://img.zhoubg.cn/static/java-volatile-2.png)



由于该变量尚未被另一个线程写回到主内存中而导致线程看不到到变量最新值的问题，被称为**“可见性”**问题。 一个线程的更新对其他线程不可见。

# `volatile`的可见性保证
Java中 ` volatile`关键字旨在解决变量可见性问题。 通过声明`counter`变量为`volatile`，所有对`counter`变量的写操作将立即写回到主存储器； 同样，对`counter`变量的所有读取将直接从主存储器读取。

变量的声明只需要在前面加上`volatile`关键字：

```java
public class SharedObject {

    public volatile int counter = 0;

}
```

这样将变量声明为`volatile`就可以保证一个线程对变量写入时对其他线程可见。

在上面给出的方案中，一个线程（T1）修改了`counter`，而另一个线程（T2）读取了`counter`（但从未修改过），只要将`counter`变量声明为`volatile`就足以保证T2对写入`counter`变量的可见性。

但是，如果T1和T2都在增加`counter`变量，那么仅声明计数器变量为`volatil`e是不够的，这后面再说。

# 完整的`volatile`可见性保证

实际上，Java `volatile`的可见性保证超出了`volatile`变量本身。其他可见性保证如下：

- 如果线程A写入`volatile`变量，并且线程B随后读取相同的`volatile`变量，则在写入`volatile`变量之前线程A可见的所有变量在读取`volatile`变量后也将对线程B可见。
- 如果线程A读取了`volatile`变量，则在读取`volatile`变量时线程A可见的所有所有变量也将从主内存中重新读取。

让我用一个代码示例来说明这一点：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

`udpate（）`方法写入了三个变量，其中只有`days`是声明了`volatile`的。

完整的`volatile`的可见性保证意味着，当将一个值写入`days`时，线程可见的所有变量也将写入主内存， 这意味着，当将值写入`days`时，`years`和`months`的值也将写入主存储器。

在读取`years`，`months`和`days`的值时，您可以这样：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public int totalDays() {
        int total = this.days;
        total += months * 30;
        total += years * 365;
        return total;
    }

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

请注意，`totalDays（）`方法将首先把`days`的值读入`total`变量， 当读取`days`的值时，`months`和`years`的值也被读入主存储器。 因此，可以保证按照上述读取顺序查看`days`，`months`和`years`的最新值。

# 指令重排的难题
出于性能原因，允许Java 虚拟机和CPU对程序中的指令进行重新排序，只要指令的语义含义保持不变即可。 例如以下代码：

```java
int a = 1;
int b = 2;

a++;
b++;
```

这些指令可以重新排序为以下顺序，而不会丢失程序的语义：

```java
int a = 1;
a++;

int b = 2;
b++;
```

然而，当变量之一是`volatile`变量时，指令重排就会产生些问题了。 让我们看一下前面的示例中的`MyClass`类：

```java
public class MyClass {
    private int years;
    private int months
    private volatile int days;

    public void update(int years, int months, int days){
        this.years  = years;
        this.months = months;
        this.days   = days;
    }
}
```

一旦`update（）`方法将值写入`days`，则对`years`和`months`新写入的值也写入主内存。 但是，如果Java虚拟机对指令进行重排，例如： 

```java
public void update(int years, int months, int days){
    this.days   = days;
    this.months = months;
    this.years  = years;
}
```

当修改`days`变量时，`months`和`years`的值仍会写入主存储器，但是这一次写入主存储器的值是在将新值写入`months`和`years`之前的那个值， 因此，新值无法被其他线程看到（不可见）。这种情况重新排序的指令的语义已经发生更改了。

当然Java有解决此问题的方法，我们将在下一节中看到。

# `Happen-Before`原则
为了解决指令重新排序的难题，除了可见性保证之外，`volatile`关键字还提供了“`Happen-Before`”保证：

- 如果对易变变量的写操作最初发生在对`volatile`变量的写操作之前，则对其他变量的读和写操作不能在写`volatile`变量后重新排序。
  确保在对`volatile`变量进行写之前进行的读/写操作在“对`volatile`变量进行写之前”发生。请注意，例如读/写位于对volatile的写入之后的其他变量，以重新排序发生在对该`volatile`的写入之前。并非相反。从后到前是允许的，但从前到后是不允许的。
- 如果读取/写入最初发生在读取` volatile`变量之后，则对其他变量的读取和写入不能重新排列为在读取`volatile`变量之前发生。请注意，可能会在读取`volatile`变量之后重新排列读取`volatile`变量之前发生的其他变量。并非相反。允许从之前到之后，但不允许从之后到之前。
  上述“先发生后保证”确保强制执行`volatile`关键字的可见性保证。

# 挥发性并不总是足够的
即使volatile关键字保证直接从主存储器读取所有volatile变量的读操作，并且将对volatile变量的所有写操作直接写到主存储器，在某些情况下，仍不足以声明volatile变量。

在前面说明的情况下，只有线程1写入共享计数器变量，声明计数器变量volatile足以确保线程2始终看到最新的写入值。

实际上，如果写入变量的新值不依赖于先前的值，则多个线程甚至可能正在写入一个共享的volatile变量，并且仍将正确的值存储在主存储器中。换句话说，如果线程首先将值写入共享的volatile变量，则不需要先读取其值即可找出下一个值。

一旦线程需要首先读取volatile变量的值，并基于该值为共享的volatile变量生成新值，则volatile变量将不再足以保证正确的可见性。读取volatile变量与写入新值之间的时间间隔很短，这造成了竞争状态，其中多个线程可能会读取volatile变量的相同值，为该变量生成一个新值，以及在写入值时返回主内存-覆盖彼此的值。

多个线程递增同一计数器的情况恰好是volatile变量不足的情况。以下各节将更详细地说明这种情况。

想象一下，如果线程1将一个值为0的共享计数器变量读入其CPU高速缓存中，将其递增为1，而不将更改后的值写回到主存储器中。然后，线程2可以从主存储器（该变量的值仍为0）读取相同的计数器变量到其自己的CPU高速缓存中。然后线程2还可将计数器增加到1，也不会将其写回到主存储器。下图说明了这种情况：

![Two threads have read a shared counter variable into their local CPU caches and incremented it.](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-3.png)

现在，线程1和线程2实际上不同步。 共享计数器变量的实际值应该为2，但是每个线程在其CPU高速缓存中的变量值均为1，而在主内存中该值仍为0。真是一团糟！ 即使线程最终将共享计数器变量的值写回到主内存，该值也将是错误的。

# 什么时候挥发足够？
如前所述，如果两个线程都在读取和写入共享变量，那么仅使用volatile关键字是不够的。在这种情况下，您需要使用同步来保证变量的读写是原子的。读取或写入volatile变量不会阻止线程读取或写入。为此，您必须在关键部分周围使用synced关键字。

作为同步块的替代方法，您还可以使用java.util.concurrent包中提供的许多原子数据类型之一。例如，AtomicLong或AtomicReference或其他之一。

如果只有一个线程读取和写入易失变量的值，而其他线程仅读取该变量，则保证读取线程可以看到写入易失变量的最新值。如果不使变量可变，则将无法保证。

volatile关键字保证可以在32位和64个变量上使用。

# `volatile`的性能考虑

读取和写入`volatile`变量会使该变量被读取或写入主存储器。与访问CPU高速缓存相比，读写主存储器的开销更大。访问`volatile`变量还可以防止指令重新排序，这是正常的性能增强技术。因此，仅在确实需要增强变量的可见性时才应使用`volatile`变量。