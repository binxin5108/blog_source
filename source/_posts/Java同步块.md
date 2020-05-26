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

我们甚至可以在Java `Lambda`表达式以及匿名类中使用同步块。

下面是是一个内部包含同步块的 `lambda`表达式的示例。 注意，`lambda`表达式中同步块是同步在的`class`对象上， 当然也可以在另一个对象上进行同步，如果这样做更有意义的话（考虑到特定的用例），但是在本示例中使用`class`对象是可以的。

```java
import java.util.function.Consumer;

public class SynchronizedExample {

  public static void main(String[] args) {

    Consumer<String> func = (String param) -> {

      synchronized(SynchronizedExample.class) {

        System.out.println(Thread.currentThread().getName() 
                           + " step 1: " + param);

        try {
          Thread.sleep( (long) (Math.random() * 1000));
        } catch (InterruptedException e) {
          e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() +
                    " step 2: " + param);
      }

    };
      

    Thread thread1 = new Thread(() -> {
        func.accept("Parameter");
    }, "Thread 1");

    Thread thread2 = new Thread(() -> {
        func.accept("Parameter");
    }, "Thread 2");

    thread1.start();
    thread2.start();
  }
}
```

# Java 同步例子

下面给出一个示例，它启动2个线程，并让它们两个都在`Counter`的同一实例上调用`add`方法。 一次仅一个线程将能够在同一实例上调用`add`方法，因为该方法在它所属的实例上是同步的。

```java
  public class Example {

    public static void main(String[] args){
      Counter counter = new Counter();
      Thread  threadA = new CounterThread(counter);
      Thread  threadB = new CounterThread(counter);

      threadA.start();
      threadB.start();
    }
  }
```

这是上面示例中使用到的两个类，`Counter`和`CounterThread`。

```java
  public class Counter{
     
     long count = 0;
    
     public synchronized void add(long value){
       this.count += value;
     }
  }
```

```java
  public class CounterThread extends Thread{

     protected Counter counter = null;

     public CounterThread(Counter counter){
        this.counter = counter;
     }

     public void run() {
         for(int i=0; i<10; i++){
             counter.add(i);
         }
     }
  }
```
分别创建了两个线程， 并使用了相同的`Counter`实例作为其构造函数的参数。因为`add`方法是实例同步方法，`Counter.add（）`方法在实例上是同步的。 因此，一次只有一个线程可以调用`add（）`方法， 另一个线程将等到第一个线程离开`add（）`方法之后才能执行该方法。

如果两个线程引用了两个单独的`Counter`实例，则同时调用`add（）`方法将没有问题。 调用将针对不同的对象，因此调用的方法也将在不同的对象（拥有该方法的对象）上同步， 因此调用不会被阻塞。 看起来是这样的：

```java
public class Example {

  public static void main(String[] args){
    Counter counterA = new Counter();
    Counter counterB = new Counter();
    Thread  threadA = new CounterThread(counterA);
    Thread  threadB = new CounterThread(counterB);

    threadA.start();
    threadB.start();
  }
}
```

请注意，线程A和线程B这两个线程不再引用相同的`Counter`实例。 `counterA`和`counterB`的`add()`方法在它们两个所属的实例上同步。 因此，在`counterA`上调用`add（）`不会阻止`counterB`对`add（）`的调用。

# 同步和数据可见性

如果不使用`synchronized`关键字（或`volatile`关键字），则无法保证当一个线程更改了与其他线程共享的变量的值时（例如所有线程都可以访问的对象），其他线程能看到更改后的值；无法保证一个线程何时将保留在CPU寄存器中的变量写回到主存储器中，也无法保证其他线程何时从主存储器“刷新” 变量的值到CPU寄存器中。

``synchronized``关键字可以改变这一点， 当线程进入同步块时，它将刷新该线程可见的所有变量的值； 当线程退出同步块时，对该线程可见的变量的所有更改都将写回给主内存。 这类似于`volatile`关键字的工作方式。

# 同步和指令重排

Java编译器和Java虚拟机允许对代码中的指令进行重新排序，以使它们更快地执行，通常是通过把指令重新排序然后由CPU并行执行来实现。指令重新排序可能会导致多个线程同时执行的代码出现问题。 例如，如果对发生在同步块内部的变量的写操作重新排序，写操作最终执行可能发生在同步块外部。

为了解决此问题，Java 对```synchronized```关键字修饰的同步块做了一些限制，对进入同步块之前、之中和之后的指令重新排序设置了一些限制。 这类似于`volatile`关键字所设置的限制。

最终结果是，您可以确保您的代码正确运行，不会发生指令重新排序而导致最终该代码的行为不同于您编写的代码所期望的行为。

# 到底在哪些对象上同步



# 同步块的限制和替代品



# 同步块的性能开销



# 同步块重入



# 集群中的同步块

