# $Java$

[TOC]

## Architecture

<img src="../assets/k8uq1.png" alt="img" style="zoom:30%;" />

- **Java Development Kit**
- **Java Runtime Environment**



- javac：用于编译java源代码，生成class文件；
- javap：用于反编译，根据class文件，反解析出其中的汇编指令和其他信息；
- javadoc：用于生成java文档的命令。

## Object Oriented Programming

## Object Memory

<img src="../assets/19c0f3d62c6874cf11707460cad2ac5242c029.png" alt="重磅硬核|一文聊透对象在JVM中的内存布局等（一）-开源基础软件社区" style="zoom:33%;" />

- 对象头

  - **mark word**: Usually a set of bitfields including synchronization state and identity hash code. May also be a pointer (with characteristic low bit encoding) to synchronization related information. During GC, may contain GC state bits.

    32 bit:

    <img src="./assets/1162587-20200918154115022-312986152.png" alt="img" style="zoom:25%;" />

    64 bit:

    <img src="./assets/1162587-20200918154125385-1537793659.png" alt="img" style="zoom:23%;" />

  - **klass pointer**: Points to another object (a meta object) which describes the layout and behavior of the original object. For Java objects, the "klass" contains a C++ style "vtable".

- **对齐数据**: 。默认情况下，Java虚拟机堆中对象的起始地址需要对齐至8的倍数。如果一个对象用不到8N个字节则需要对其填充，以此来补齐对象头和实例数据占用内存之后剩余的空间大小。如果对象头和实例数据已经占满了JVM所分配的内存空间，那么就不用再进行对齐填充了。

  - 字段内存对齐的其中一个原因，是让字段只出现在同一CPU的缓存行中。如果字段不是对齐的，那么就有可能出现跨缓存行的字段。也就是说，该字段的读取可能需要替换两个缓存行，而该字段的存储也会同时污染两个缓存行。这两种情况对程序的执行效率而言都是不利的。其实对其填充的最终目的是为了计算机高效寻址。

## Grammar

### Annotate

- meta annotation

  - `@Target`: is used to specify at which type, the annotation is used.

    `TYPE, FIELD, METHOD, CONSTRUCTOR, LOCAL_VARIABLE, ANNOTATION_TYPE, PARAMETER`.

  - `@Retention`: is used to specify to what level annotation will be available.

    <img src="./assets/v2-2ed053ec88374f582494338815a0886c_720w.webp" alt="img" style="zoom:50%;" />

    - `RetentionPolicy.SOURCE`
    - `RetentionPolicy.CLASS`
    - `RetentionPolicy.RUNTIME`

  - `@Documented`: Marks the annotation for inclusion in the documentation.

  - `@Inherited`: marks the annotation to be inherited to subclasses.

1. 定义注解
2. 使用注解
3. 读取并执行相应流程

Define

```java
public @interface Annotation_name {
    attribute list;
}
```



## Compilation

<img src="./assets/640-1692207008579-8.jpeg" alt="Image" style="zoom:50%;" />

<img src="./assets/640-1692207651527-17.jpeg" alt="Image" style="zoom:33%;" />

**1. 词法分析&语法分析**

<img src="./assets/640-1692207966107-20.jpeg" alt="Image" style="zoom: 40%;" />



<img src="./assets/640-1692208030219-23.jpeg" alt="Image" style="zoom: 40%;" />

**2. 填充符号表**



**3. 注解处理**

并不是所有的注解都是在编译期起作用的，我们平时用反射处理的注解主要是指运行时注解

**4. 语义分析**

语义分析的功能就是从结构和规则上对源代码进行检查，包括声明检查和类型检查等等。

4.1 标注检查

泛型方法类型的推导, 常量折叠（Constant Folding）

4.2 数据流分析

数据流分析是在标注检查之后的进一步检验，主要检验是局部变量在使用前是否确定性赋值、声明有返回值的方法是否有确定性的返回值等。值得注意的是，final变量不可重复赋值的性质也是在这一步检查，如果一个final变量被重复赋值，编译器会发现并报错的。

**5. 解语法糖**

java中的自动拆箱装箱功能、foreach循环功能

**6. 生成Class文件**

构建的语法树、符号表等信息，在这一步被转换成字节码指令写到class文件中

## JVM Class Loading

### Class Loader

- Bootstrap ClassLoader
- Extension ClassLoader
- Application ClassLoader

<img src="./assets/640-1692212310530-29.jpeg" alt="Image" style="zoom: 33%;" />

<img src="./assets/640-1692212339155-32.jpeg" alt="Image" style="zoom:33%;" />

### Parent Delegation Mechanism

双亲委托，是当前类加载器(以系统类加载器为例)在加载一个类时，委托给其双亲（注意这里的双亲指的是类加载器中parent属性指向的类加载器）先进行加载。

<img src="./assets/640-1692212517546-35.jpeg" alt="Image" style="zoom:33%;" />

### 打破

- JDBC

  你先得知道SPI(Service Provider Interface)，这玩意和API不一样，它是面向拓展的，也就是我定义了这个SPI，具体如何实现由扩展者实现。我就是定了个规矩。

  JDBC就是如此，在rt.jar里面定义了这个SPI，那mysql有mysql的jdbc实现，oracle有oracle的jdbc实现，反正我java不管你内部如何实现的，反正你们都得统一按我这个来，这样我们java开发者才能容易的调用数据库操作。所以因为这样那就不得不违反这个约束啊，`Bootstrap ClassLoader`就得委托子类来加载数据库厂商们提供的具体实现。因为它的手只能摸到`<JAVA_HOME>\lib`中，其他的它无能为力。这就违反了自下而上的委托机制了。

  Java就搞了个线程上下文类加载器，通过`setContextClassLoader()`默认情况就是应用程序类加载器然后`Thread.current.currentThread().getContextClassLoader()`获得类加载器来加载。



### 类连接

类在加载进来之后，会进行连接、初始化，最后才会被使用。在连接过程中，又包括验证、准备和解析三个部分。

**验证：**验证类符合 Java 规范和 JVM 规范，在保证符合规范的前提下，避免危害虚拟机安全。

**准备：**为类的静态变量分配内存，初始化为系统的初始值。对于 final static 修饰的变量，直接赋值为用户的定义值。例如，private final static int value=123，会在准备阶段分配内存，并初始化值为 123，而如果是 private static int value=123，这个阶段 value 的值仍然为 0。

**解析：**将符号引用转为直接引用的过程。我们知道，在编译时，Java 类并不知道所引用的类的实际地址，因此只能使用符号引用来代替。类结构文件的常量池中存储了符号引用，包括类和接口的全限定名、类引用、方法引用以及成员变量引用等。如果要使用这些类和方法，就需要把它们转化为 JVM 可以直接获取的内存地址或指针，即直接引用。

### 类初始化

类初始化阶段是类加载过程的最后阶段，在这个阶段中，JVM 首先将执行构造器 方法，编译器会在将 .java 文件编译成 .class 文件时，收集所有类初始化代码，包括静态变量赋值语句、静态代码块、静态方法，收集在一起成为 () 方法。

## 即时编译

初始化完成后，类在调用执行过程中，执行引擎会把字节码转为机器码，然后在操作系统中才能执行。在字节码转换为机器码的过程中，虚拟机中还存在着一道编译，那就是即时编译。

最初，虚拟机中的字节码是由解释器（ Interpreter ）完成编译的，当虚拟机发现某个方法或代码块的运行特别频繁的时候，就会把这些代码认定为“热点代码”。

为了提高热点代码的执行效率，在运行时，即时编译器（JIT）会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化，然后保存到内存中。

<img src="../assets/8d4ec9c73ec37d69adb105aa7d052717.jpg" alt="img" style="zoom: 25%;" />

## Concurrent Programming

### Locks

- Synchronized Block

synchronized是java提供的原子性内置锁，这种内置的并且使用者看不到的锁也被称为监视器锁，使用synchronized之后，会在编译之后在同步的代码块前后加上monitorenter和monitorexit字节码指令，他依赖操作系统底层互斥锁实现。他的作用主要就是实现原子性操作和解决共享变量的内存可见性问题。

执行monitorenter指令时会尝试获取对象锁，如果对象没有被锁定或者已经获得了锁，锁的计数器+1。此时其他竞争锁的线程则会进入等待队列中。执行monitorexit指令时则会把计数器-1，当计数器值为0时，则锁释放，处于等待队列中的线程再继续竞争锁。

<img src="./assets/640-1692209451980-26.jpeg" alt="Image" style="zoom:50%;" />

- 



锁的状态从低到高依次为无锁->偏向锁->轻量级锁->重量级锁，升级的过程就是从低到高，降级在一定条件也是有可能发生的。

## 实现多线程


在Java中，编写多线程程序通常可以通过两种方式来实现：继承Thread类和实现Runnable接口。以下是这两种方法的示例代码：

**方法一：继承Thread类**

```
javaCopy codeclass MyThread extends Thread {
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("Thread " + Thread.currentThread().getId() + ": " + i);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread1 = new MyThread();
        MyThread thread2 = new MyThread();

        thread1.start();
        thread2.start();
    }
}
```

**方法二：实现Runnable接口**

```
javaCopy codeclass MyRunnable implements Runnable {
    public void run() {
        for (int i = 0; i < 10; i++) {
            System.out.println("Thread " + Thread.currentThread().getId() + ": " + i);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyRunnable runnable = new MyRunnable();

        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);

        thread1.start();
        thread2.start();
    }
}
```

无论使用哪种方法，都是创建一个新的线程并调用其 `start()` 方法来启动线程。每个线程会执行 `run()` 方法中的代码。多线程的执行顺序和调度是由操作系统和JVM控制的，所以输出结果可能在不同次运行中会有所不同。

使用Executor框架、Callable接口和线程池可以更加有效地管理和控制多线程任务。这些方法可以提高多线程程序的性能和可维护性。以下是示例代码来演示如何使用这些技术：

**使用Executor框架和Callable接口：**

```
javaCopy codeimport java.util.concurrent.*;

class MyCallable implements Callable<Integer> {
    public Integer call() {
        int sum = 0;
        for (int i = 1; i <= 10; i++) {
            sum += i;
        }
        return sum;
    }
}

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Callable<Integer> callable = new MyCallable();

        Future<Integer> future = executor.submit(callable);

        try {
            Integer result = future.get();
            System.out.println("Result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        } finally {
            executor.shutdown();
        }
    }
}
```

**使用线程池：**

```
javaCopy codeimport java.util.concurrent.*;

class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Thread " + Thread.currentThread().getId() + " is executing.");
    }
}

public class Main {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2); // 创建一个拥有2个线程的线程池

        Runnable task1 = new MyRunnable();
        Runnable task2 = new MyRunnable();
        Runnable task3 = new MyRunnable();

        executor.execute(task1);
        executor.execute(task2);
        executor.execute(task3);

        executor.shutdown(); // 关闭线程池
    }
}
```

在上述代码中，使用了Executor框架来调度任务的执行。`Callable` 接口可以返回一个结果，而 `Runnable` 接口则不能。使用 `ExecutorService` 可以管理线程池，创建一个可以管理和调度线程的池子。在使用完毕后，记得通过 `shutdown()` 方法关闭线程池。

线程池的优点在于可以复用线程，避免频繁创建和销毁线程的开销，同时可以控制并发线程的数量，以防止系统资源耗尽。Callable接口则允许线程返回一个结果，在任务完成后可以通过 `Future` 对象获取结果。

## Thread Local

## [JVM (Java Virtual Machine)](./Java_Virtual_Machine.md)

