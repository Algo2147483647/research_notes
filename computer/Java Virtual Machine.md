# JVM (Java Virtual Machine)

[TOC]

- Heap
- String Constant Pool

## Memory Model

<img src="assets/java-runtime-data-areas-jdk1.8.png" alt="Java 运行时数据区域（JDK1.8 ）" style="zoom: 50%;" />

### Heap

此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。

- Young Generation
- Old Generation
- Permanent Generation

### Program counter

当前线程所执行的字节码的行号指示器。字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制。

多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

### Stack

栈由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法返回地址。

- **局部变量表** 主要存放了编译期可知的各种数据类型
- **操作数栈** 主要作为方法调用的中转站使用，用于存放方法执行过程中产生的中间计算结果。另外，计算过程中产生的临时变量也会放在操作数栈中。

## Garbage Collection

### Identify Garbage: reachability algorithm

The principle of the reachability algorithm is to start from a series of objects called GC Root, lead to the next node they point to, and then start from the next node to lead to the next node pointed to by this node. . . (In this way, a line formed by GC Root is called a reference chain), until all nodes are traversed, if the related objects are not in any reference chain starting from GC Root, these objects will be judged as "garbage" ”, will be recycled by GC.

$$
Eden:S_0:S_1 = 8:1:1
$$

promoted to the old generation

- 当对象的年龄达到了我们设定的阈值，则会从S0（或S1）晋升到老年代
- 大对象 当某个对象分配需要大量的连续内存时，此时对象的创建不会分配在 Eden 区，会直接分配在老年代，因为如果把大对象分配在 Eden 区, Minor GC 后再移动到 S0,S1 会有很大的开销（对象比较大，复制会比较慢，也占空间），也很快会占满 S0,S1 区，所以干脆就直接移到老年代.
- 在 S0（或S1） 区相同年龄的对象大小之和大于 S0（或S1）空间一半以上时，则年龄大于等于该年龄的对象也会晋升到老年代。

<img src="../assets/promoted_to_the_old_generation.gif" alt="img" style="zoom:50%;" />



<img src="../assets/java-collection-6.png.webp" alt="img" style="zoom:50%;" />

- 标记清除
- 标记整理
- 复制算法

### CMS

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

### G1 (*Garbage First*)

G1 (Garbage-First) 是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.

<img src="assets/640-1692206734380-5.png" alt="Image" style="zoom:50%;" />

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

## Class

### Architecture

<img src="assets/16d5ec47609818fc.jpeg" alt="ClassFile 内容分析" style="zoom: 15%;" />

- Constant Pool
- Access Flags

### life cycle

- Loading
- Linking
  - Verification
  - Preparation
  - Resolution
- Initialization
- Using
- Unloading

### Class loader

1. 通过全类名获取定义此类的二进制字节流。
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口。

类加载器是一个负责加载类的对象。`ClassLoader` 是一个抽象类。给定类的二进制名称，类加载器应尝试定位或生成构成类定义的数据。典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的“类文件”。

每个 Java 类都有一个引用指向加载它的 `ClassLoader`。不过，数组类不是通过 `ClassLoader` 创建的，而是 JVM 在需要的时候自动创建的，数组类通过`getClassLoader()`方法获取 `ClassLoader` 的时候和该数组的元素类型的 `ClassLoader` 是一致的。

类加载器的主要作用就是加载 Java 类的字节码（ .class 文件）到 JVM 中（在内存中生成一个代表该类的 Class 对象）。 字节码可以是 Java 源程序（.java文件）经过 javac 编译得来，也可以是通过工具动态生成或者通过网络下载得来。

- **`BootstrapClassLoader`(启动类加载器)**：最顶层的加载类，由 C++实现，通常表示为 null，并且没有父级，主要用来加载 JDK 内部的核心类库（ `%JAVA_HOME%/lib`目录下的 `rt.jar`、`resources.jar`、`charsets.jar`等 jar 包和类）以及被 `-Xbootclasspath`参数指定的路径下的所有类。

- **`ExtensionClassLoader`(扩展类加载器)**：主要负责加载 `%JRE_HOME%/lib/ext` 目录下的 jar 包和类以及被 `java.ext.dirs` 系统变量所指定的路径下的所有类。

- **`AppClassLoader`(应用程序类加载器)**：面向我们用户的加载器，负责加载当前应用 classpath 下的所有 jar 包和类。

<img src="assets/class-loader-parents-delegation-model.png" alt="类加载器层次关系图" style="zoom: 50%;" />

#### Parental delegation

`ClassLoader` 类使用委托模型来搜索类和资源。每个 `ClassLoader` 实例都有一个相关的父类加载器。需要查找类或资源时，`ClassLoader` 实例会在试图亲自查找类或资源之前，将搜索类或资源的任务委托给其父类加载器。
 虚拟机中被称为 "bootstrap class loader"的内置类加载器本身没有父类加载器，但是可以作为 `ClassLoader` 实例的父类加载器。

在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载（每个父类加载器都会走一遍这个流程）。

类加载器在进行类加载的时候，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成（调用父加载器 `loadClass()`方法来加载类）。这样的话，所有的请求最终都会传送到顶层的启动类加载器 `BootstrapClassLoader` 中。

只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载（调用自己的 `findClass()` 方法来加载类）。

如果子类加载器也无法加载这个类，那么它会抛出一个 `ClassNotFoundException` 异常。

