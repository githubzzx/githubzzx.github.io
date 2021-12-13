---
title: （一）JVM Heap参数配置
date: 2021-12-13 10:41:07
tags: 
 - JVM
categories:
 - JVM
---

# <!-- more -->JVM内存模型

![heap.png](https://s2.loli.net/2021/12/13/w5YoMb6ETXNdLcV.png)

JVM内存结构主要有三大块：**堆内存**、**方法区**和**栈**

对于大多数应用来说，Java堆（Java Heap）是Java虚拟机所管理的内存中**最大**的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，**几乎所有的对象实例都在这里分配内存**。

堆内存是JVM中最大的一块由年轻代和老年代组成，而年轻代内存又被分成三部分，**Eden空间**、**From Survivor空间**、**To Survivor空间**。

# 堆（Heap）配置

> 按照官方的说法：“Java 虚拟机具有一个堆(Heap)，堆是运行时数据区域，所有类实例和数组的内存均从此处分配。堆是在 Java 虚拟机启动时创建的。”

堆区的大小，主要由以下几个参数进行控制：

1、Xms

2、Xmx

3、MaxHeapSize

4、InitalHeapSize

## Xms与InitialHeapSize

Xms等价于InitialHeapSize，表示Heap的初始化大小，即JVM启动时，堆区的最小值，使用该参数的正确姿势是：

```java
-Xms10m
-XX:InitialHeapSize=10m
```

**注意：**

> 1. InitialHeapSize的最小值是1M，如果小于这个值，JVM启动会报错
> 2. Xms与InitialHeapSize表示的含义是相同的，但是这两个参数如果同时设置，那么生效的只有最后设置的参数。

## Xmx与MaxHeapSize

Xmx等价于MaxHeapSize，表示Heap的最大值大小，即Heap区可以分配使用最大内存值，使用该参数的正确姿势是：

```java
-Xmx100m
-XX:MaxHeapSize=100m
```

**注意：**

> MaxHeapSize与InitialHeapSize的关系，MaxHeapSize是必须要大于等于InitialHeapSize的，否则JVM无法启动

## Heap的缺省配置

如果我们没有设置Heap的大小，JVM会如何设定呢？

Java8中Oracle的官方说法：

> **Default Heap Size**
> Unless the initial and maximum heap sizes are specified on the command line, they are calculated based on the amount of memory on the machine.
>
> **Client JVM Default Initial and Maximum Heap Sizes**
> The default maximum heap size is half of the physical memory up to a physical memory size of 192 megabytes (MB) and otherwise one fourth of the physical memory up to a physical memory size of 1 gigabyte (GB).
>
> For example, if your computer has 128 MB of physical memory, then the maximum heap size is 64 MB, and greater than or equal to 1 GB of physical memory results in a maximum heap size of 256 MB.
>
> The maximum heap size is not actually used by the JVM unless your program creates enough objects to require it. A much smaller amount, called the initial heap size, is allocated during JVM initialization. This amount is at least 8 MB and otherwise 1/64th of physical memory up to a physical memory size of 1 GB.
>
> The maximum amount of space allocated to the young generation is one third of the total heap size.
>
> **Server JVM Default Initial and Maximum Heap Sizes**
>
> The default initial and maximum heap sizes work similarly on the server JVM as it does on the client JVM, except that the default values can go higher. On 32-bit JVMs, the default maximum heap size can be up to 1 GB if there is 4 GB or more of physical memory. On 64-bit JVMs, the default maximum heap size can be up to 32 GB if there is 128 GB or more of physical memory. You can always set a higher or lower initial and maximum heap by specifying those values directly; see the next section.

根据Oracle官方文档的说法，JVM的默认堆大小如果未指定，它将会根据服务器物理内存计算而来的。

**client模式下，JVM初始和最大堆大小为：**
在物理内存达到192MB之前，JVM最大堆大小为物理内存的一半，否则，在物理内存大于192MB，在到达1GB之前，JVM最大堆大小为物理内存的1/4，大于1GB的物理内存也按1GB计算，举个例子，如果你的电脑内存是128MB，那么最大堆大小就是64MB，如果你的物理内存大于或等于1GB，那么最大堆大小为256MB。
Java初始堆大小是物理内存的1/64，但最小是8MB。

**server模式下：**
与client模式类似，区别就是默认值可以更大，比如在32位JVM下，如果物理内存在4G或更高，最大堆大小可以提升至1GB，，如果是在64位JVM下，如果物理内存在128GB或更高，最大堆大小可以提升至32GB。

# 堆（Heap）的动态调整

JVM启动之后，整个Heap的大小是固定的。但不代表整个堆里的内存都可用。在GC之后会根据一些参数进行动态调整，比如Xms和Xmx不一样的时候，表示堆里的新生代和老生代的可用内存在不断的变化，所以提出了一个概念: **相关堆的有效内存** ，**有效内存表示真正可用的内存**

一般情况下都会将初始内存（-Xms）和 最大内存（-Xmx）的值设置一样大小，理论上堆的大小是恒定的。这样可以减少GC的次数。

但是稳定的堆，虽然可以减少GC的次数，但同时也增加了每次GC的时间。在系统不需要那么大的内存时，可以压缩堆的空间，加快单次GC的速度。基于这样的考虑，JVM提出两个参数用于压缩和扩展堆的空间。

> -XX:MinHeapFreeRatio 参数用来设置堆空间最小空闲比例，默认值是 40。当堆空间的空闲内存小于这个数值时，JVM 便会扩展堆空间。

> -XX:MaxHeapFreeRatio 参数用来设置堆空间最大空闲比例，默认值是 70。当堆空间的空闲内存大于这个数值时，便会压缩堆空间，得到一个较小的堆。

**注意：**

> 1. 当-Xmx 和-Xms 相等时，-XX:MinHeapFreeRatio 和-XX:MaxHeapFreeRatio 两个参数无效。
> 2. 在实际生产环境中，如果不是对JVM Heap的情况极为了解，不建议设置该参数。
> 3. 仍是建议将-Xms与-Xmx的大小设置为一样值，避免Heap的大小扩张缩小。