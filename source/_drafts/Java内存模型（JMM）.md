---
title: Java内存模型（JMM）
tags:
  - Java并发
categories: []
toc: false
---

Java内存模型描述了Java虚拟机如何在计算机的内存（RAM）中工作。 Java虚拟机其实是个计算机的模型，因此该模型自然包含一个内存模型-那就是Java内存模型。

如果要设计出行为表现正确的并发程序，就必须了解Java内存模型。 Java内存模型描述了不同线程如何以及何时可以看到被其他线程写入到共享变量的值，以及在必要时如何同步对共享变量的访问。

原始的Java内存模型存在着欠缺，因此在Java 1.5版本中对Java内存模型进行了修订。 此版本的Java内存模型仍在Java 8中继续使用。

# Java内存模型

`JVM`内部将内存分为线程堆栈（`thread stacks`）和堆(`heap`)。下图从逻辑角度阐明了Java内存模型：
![image.png](http://img.zhoubg.cn/FrxpxZG5pb-X_flvRVdawLWOBIDH)

Java虚拟机中运行的每个线程都有其自己的线程堆栈。线程堆栈包含了当前线程调用了哪些方法以及当前执行点的信息。我们将其称为“调用堆栈”。当线程执行其代码时，调用堆栈也随之改变。

线程堆栈还包含正在执行的每个方法（调用堆栈上的所有方法）的所有局部变量。每个线程只能访问自己的线程堆栈。由线程创建的局部变量对其他的线程是不可见的。即使两个线程执行的代码完全相同，这两个线程仍将在各自的线程堆栈中创建该代码的局部变量。因此，每个线程对每个局部变量都有其自己的版本。

所有原始类型的局部变量（ `boolean, byte, short, char, int, long, float, double`）都完全存储在线程堆栈中，对其他线程是不可见。一个线程可以将一个原始类型的变量副本传递给另一个线程，但是它不能共享变量本身。

堆里面包含了Java应用程序中创建的所有对象，不管创建对象的线程是什么。这其中包括了基本数据类型的包装对象（例如Byte，Integer，Long等）。不管一个对象是作为方法的局部变量或者是类的成员变量，该对象本身仍然是存储在堆中的。

说明了调用堆栈和存储在线程堆栈上的局部变量以及存储在堆上的对象的关系图：

![image.png](http://img.zhoubg.cn/FnMVOeHXm3nfgwAWN94SWIAqjVbl)

局部变量可以是原始类型，在这种情况下，它完全保留在线程堆栈中。

局部变量也可以是对对象的引用。在这种情况下，引用（局部变量）存储在线程堆栈中，但是对象本身存储在堆中。

一个对象可能包含方法，而这些方法可能包含局部变量。虽然这些方法所属的对象存储在堆中，但这些局部变量还是存储在线程堆栈中。

对象的成员变量与对象本身一起存储在堆中，不管成员变量是原始类型还是对对象的引用。

静态类变量也与类定义一起存储在堆中。

所有线程都可以通过对对象的引用访问堆上的对象。当线程可以访问某个对象时，它也可以访问该对象的成员变量。如果两个线程同时在同一个对象上调用方法，则它们都将有权访问该对象的成员变量，但是每个线程将拥有自己的局部变量副本。

这是说明以上几点的图示：

![image.png](http://img.zhoubg.cn/FgiKBonGjPNtXPF4PD43oeFKHBoh)

两个线程都具有一组局部变量。局部变量`Local variable 2`指向堆上的共享对象`Object3`。这两个线程分别具有对同一对象的不同引用。它们的引用是局部变量，因此存储在每个线程的自己的线程堆栈中。但是，两个不同的引用都指向存储在堆上的同一对象。

请注意，共享对象`Object3`拥有两个成员变量，他们是引用类型，分别指向了对象`Object2` 和 对象`Object4`（如图中从`Object3`到`Object2`和`Object4`的箭头所示）。通过`Object3`中的这些成员变量引用，两个线程就可以访问`Object2`和`Object4`。

该图中还显示了一个局部变量`Local variable 1`，该局部变量分别指向堆上的两个不同对象`Object1`和`Object5`,而不是同一个对象。理论上，当两个线程都引用了两个对象时，则两个线程都可以访问对`Object1`和对`Object5`。但是在上图中，每个线程仅具有对两个对象之一的引用,所有他们只能访问各自引用的对象。

那么，哪种Java代码可能实现上面的内存图？好吧，看如下的代码，是不是很简单：
```java
public class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 = MySharedObject.sharedInstance;
            
        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}
```
```java
public class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance = new MySharedObject();

    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);
	
    //member variables of primary type
    
    public long member1 = 12345;
    public long member2 = 67890;
}
```
如果有两个线程正在执行`run（）`方法，那么前面显示的图就是结果。 `run（）`方法调用`methodOne（）`，`methodOne（）`调用`methodTwo（）`。

`methodOne（）`声明一个原始的局部变量（int类型的`localVariable1`）和一个作为对象引用的局部变量（`localVariable2`）。

每个执行`methodOne（）`的线程将在各自的线程堆栈上创建自己的`localVariable1`和`localVariable2`副本。 `localVariable1`变量将完全的彼此分离，仅存在于每个线程的线程堆栈中。一个线程看不到另一个线程对其`localVariable1`副本所做的更改。

每个执行`methodOne（）`的线程还将创建自己的`localVariable2`副本。但是，`localVariable2`的两个不同副本最终都指向堆上的同一对象。该代码将`localVariable2`设置为静态变量,变量引用指向的对象只有一个副本，并且此副本存储在堆中。因此，`localVariable2`的两个副本最终都指向静态变量引用的`MySharedObject`的同一实例。 `MySharedObject`实例也存储在堆中。它对应于上图中的`Object3`。

注意`MySharedObject`类也包含两个成员变量。成员变量本身与对象一起存储在堆中。这两个成员变量指向另外两个`Integer`对象。这些整数对象对应于上图中的`Object2`和`Object4`。

还要注意`methodTwo（）`如何创建一个名为`localVariable1`的局部变量。此局部变量是对Integer对象的对象引用。该方法将`localVariable1`引用设置为指向新的Integer实例。执行`methodTwo（）`的每个线程的`localVariable1`引用将存储在一个副本中。实例化的两个Integer对象将存储在堆上，但是由于该方法每次执行该方法时都会创建一个新的`Integer`对象，因此执行此方法的两个线程将创建单独的Integer实例。在`methodTwo（）`内部创建的Integer对象对应于上图中的`Object1`和`Object5`。

还请注意类`MySharedObject`中的两个成员变量，其类型为`long`，这是原始类型。由于这些变量是成员变量，因此它们仍与对象一起存储在堆中。仅局部变量存储在线程堆栈上。

# 硬件内存结构

现代硬件内存体系结构与虚拟机内部的Java内存模型有所不同。了解硬件内存架构并了解Java内存模型如何与之协同工作同样也很重要。 本节将描述常见的硬件内存体系结构，下一节将描述Java内存模型如何与之协同工作。

这是现代计算机硬件体系结构的简化图：

![image.png](http://img.zhoubg.cn/Fiw4fKT3Hkfg1fVU4PfmYuG3539r)

现代计算机通常其中装有2个或更多的CPU，其中一些CPU也可能具有多个内核。关键是，在具有2个或更多CPU的现代计算机上，可能同时运行多个线程。每个CPU都可以在任何给定时间运行一个线程。这意味着，如果Java应用程序是多线程的，则每个CPU可能在Java应用程序中同时（并发）运行一个线程。

每个CPU包含一组寄存器，这些寄存器本质上是CPU内存。 CPU在这些寄存器上执行操作的速度比对主存储器中的变量执行操作的速度快得多。这是因为CPU可以比访问主存储器更快地访问这些寄存器。

每个CPU可能还具有一个CPU高速缓存存储层。实际上，大多数现代CPU都具有一定大小的缓存层。 CPU访问其高速缓存层比访问主存储器更快，但是通常不如访问寄存器的速度快。因此，CPU高速缓存内存访问速度介于寄存器和主内存的速度之间。某些CPU可能具有多个高速缓存层（1级缓存和2级缓存），但是了解Java内存模型如何与内存交互并不重要。重要的是要知道CPU可以具有某种高速缓存层。

计算机还包含一个主存储区（RAM），所有CPU都可以访问主存储器，主存储区通常比CPU的缓存大得多。

通常，当CPU需要访问主存储器时，它将部分主存储器内容读入其CPU高速缓存中，它甚至可以将缓存的一部分内容读入其内部寄存器，然后对其执行操作。当CPU需要将结果写回主存储器时，它将把值从其内部寄存器刷新到高速缓存，并在某个时候将值刷新回主存储器。

当CPU需要将其他内容存储在高速缓存中时，通常会将高速缓存中已经存储的值刷新回主存储器。 CPU高速缓存可以一次将数据写入其部分缓存空间，并一次刷新其部分内存。它不必每次更新都读取/写入全部的缓存，通常，缓存在称为“缓存行”的较小存储块中更新，可以将一个或多个高速缓存行读入高速缓存存储器，并且可以将一个或多个高速缓存行再次刷新回主存储器。

# Java内存模型和硬件内存结构的关系

如前所述，Java内存模型和硬件内存体系结构是不同的， 硬件内存体系结构不区分线程堆栈和堆。 在硬件上，线程堆栈和堆都位于主内存中；有时，部分线程堆栈和堆可能会出现在CPU缓存和内部CPU寄存器中。 下图对此进行了说明：

![image.png](http://img.zhoubg.cn/FvN7YbRp9nIkK3I2aNRJuusCMr8S)

当对象和变量可以存储在计算机的各种不同存储区域中时，就可能会出现某些问题了，两个主要问题是：

- 线程更新（写入）到共享变量的可见性。
- 读取，检查和写入共享变量时的竞争条件。

这两个问题将在以下各节中进行说明。

## 共享对象的可见性

如果两个或多个线程共享一个对象，并且没有正确使用`volatitle`声明或同步，则一个线程对共享对象进行的更新可能对其他线程不可见。

想象一下，共享对象最初存储在主存储器中，然后，在CPU上运行的一个线程将共享对象读入其CPU缓存中，在那里，它更改了共享对象。只要未将CPU高速缓存刷新回主内存，在其他CPU上运行的线程就看不到共享对象更改后的版本。这样，每个线程都可以拥有自己的共享对象副本，每个副本位于不同的CPU缓存中。

下图说明了这种情况。在左CPU上运行的一个线程将共享对象复制到其CPU缓存中，并将其`count`变量值加一变成2,在右CPU上运行的其他线程看不到此更改，因为左线程还未将··更新刷新回主存储器中。

![image.png](http://img.zhoubg.cn/Fh3rnLlrYdxSR685Nckw1Z3J5d5U)

要解决此问题，您可以使用Java的`volatile`关键字, `volatile`关键字可以确保给定的变量直接从主存储器中读取，并在更新时始终写回到主存储器中。

## 竞争条件
如果两个或多个线程共享一个对象，并且有多个线程更新该共享对象中的变量，则可能会发生竞争条件。

想象一下，线程A将共享对象的变量`count`读入其CPU缓存中。还要想象一下，线程B的功能相同，但是它位于不同的CPU缓存中。现在线程A给`count`加1，线程B执行相同的操作，现在`count`已经加了两次，在每个CPU高速缓存中分别加了一次。

如果这些增加是顺序执行的，则变成`count`将增加两次，并将原始值+ 2写回到主存储器。

但是，这两个增量是在没有适当同步的情况下同时执行的，不管线程A和B中哪个线程将其更新后的`count`写回主存中，虽然有两个增量，但更新后的值仅比原始值大1。

该图说明了如上所述的竞争条件问题的发生：

![image.png](http://img.zhoubg.cn/FvB3aSSP2ql2ct0OoB_GJVkqqHFz)

要解决此问题，您可以使用Java同步块（`synchronized block`）。 同步块可确保在任何给定时间只有一个线程可以进入代码的给定关键部分。 同步块还保证从同步块中读取的所有变量都从主存储器中读取，并且当线程退出同步块时，所有更新的变量将再次刷新回主存储器，不管该变量有没有声明为`volatile`。