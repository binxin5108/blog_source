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

# 实例方法中代码同步块
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

# 静态方法中代码同步块

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

# 在哪些对象上同步？

我们多次提到了，同步块必须在某个对象上同步，实际上，您可以选择任何的对象，但是建议您不要在`String`对象或任何原始类型包装对象上进行同步，因为编译器可能会优化这些对象，以便在您的不同位置使用相同的实例，您以为您正在使用其他实例，但其实可能是同一个实例。看这个例子：

```java
synchronized("Hey") {
   //do something in here.
}
```

如果您有多个在字面量`String`值`"hey"`同步的同步块，则编译器实际上可能在幕后使用相同的`String`对象。结果是，这两个同步块随后都在同一对象上同步,那可能不是您想要的行为。

使用原始类型包装器对象也是如此，看这个例子：

```java
synchronized(Integer.valueOf(1)) {
   //do something in here.
}
```

如果多次调用`Integer.valueOf（1）`，它实际上可能为相同的输入参数值返回相同的包装对象实例。这意味着，如果要在同一个原始包装对象上同步多个同步块（例如，将`Integer.valueOf（1）`多次用作监视对象），则会有这些同步块都在同一个对象上同步的风险，那也可能不是您想要的行为。

为了安全起见，请在`this`或`new Object（）`上进行同步， Java编译器，Java 虚拟机或Java库不会在内部对其进行缓存或重用。

# 同步块的局限性和替代方案

Java中的同步块有几个限制。例如，Java中的同步块仅一次仅允许一个线程进入，但是，如果两个线程只想读取一个共享值而不更新它，该怎么办？这种操作应该是安全的。作为同步块的替代方法，您可以使用读写锁[Read / Write Lock](http://tutorials.jenkov.com/java-concurrency/read-write-locks.html)来保护代码，该锁比同步块具有更高级的锁定语义。 Java实际上附带了您可以使用的内置[ReadWriteLock](http://tutorials.jenkov.com/java-util-concurrent/readwritelock.html)类。

如果要允许`N`个线程进入一个同步块，而不仅仅是一个线程，该怎么办？这时您可以使用信号量 [Semaphore](http://tutorials.jenkov.com/java-concurrency/semaphore.html)来实现该行为。 Java中也内置了 [Java Semaphore](http://tutorials.jenkov.com/java-util-concurrent/semaphore.html)类。

同步块不能保证等待进入线程的线程以什么顺序访问同步块。如果您需要保证尝试进入同步块的线程能够以他们请求访问的确切顺序进行访问，那该怎么办？您需要自己实现公平 [Fairness](http://tutorials.jenkov.com/java-concurrency/starvation-and-fairness.html) 机制。

如果您只有一个线程写入共享变量，而其他线程仅读取该变量怎么办？这样的话，您就可以只使用一个[volatile](http://tutorials.jenkov.com/java-concurrency/volatile.html)变量而无需进行任何同步。

# 同步块的性能开销

进入和退出同步块时会带来较小的性能开销，随着Java的发展，性能开销虽然下降了，但是仍然要付出很小的代价。

如果代码会频繁的进入和退出同步块，则通常要慎重考虑进入和退出同步块的性能开销了。

另外，请尽量使同步块的范围保持最小， 换句话说，仅同步真正需要同步的操作-避免阻止其他线程执行不必同步的操作。 同步块中只有绝对必要的指令，这样可以增加代码的并行性。

# 同步块重入

一旦线程进入同步块，我们就可以称该线程拥有了同步对象（监视对象）的锁。如果线程调用另一个方法，该方法在内部包含同步块的情况下回调第一个方法，则持有锁的线程可以重新进入同步块，因为线程（本身）持有锁而被不会阻塞，仅当其他线程持有该锁时才会阻塞。看这个例子：

```java
public class MyClass {
    
  List<String> elements = new ArrayList<String>();
    
  public void count() {
    if(elements.size() == 0) {
        return 0;
    }
    synchronized(this) {
       elements.remove();
       return 1 + count();  
    }
  }
    
}
```

暂时不用管上述计算列表元素的方法有没有意义，只需关注`count（）`方法内的同步块内部如何递归调用`count（）`方法即可。因此，线程调用`count（）`最终可能会多次进入同一同步块，这种情况是允许的。

但是请记住，如果您不仔细设计代码，则线程进入多个同步块的设计可能导致嵌套的监视器锁定[nested monitor lockout](http://tutorials.jenkov.com/java-concurrency/nested-monitor-lockout.html)。

# 集群中的同步块

请记住，同步块仅阻止同一Java虚拟机中的线程进入该代码块。 如果在集群中的多个Java 虚拟机上运行相同的Java应用程序，则每个Java 虚拟机中的一个线程可能会同时进入该同步块。

如果需要跨集群中所有Java 虚拟机进行同步，则将需要使用其他同步机制，而不仅仅是同步块。