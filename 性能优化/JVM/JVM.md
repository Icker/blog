---
title: JVM
tags: java
categories: 
- 性能优化
- JVM
---



# 什么是JVM

JVM（java virtual machine）即java虚拟机。是运行java语言的虚拟机。实现了java语言的跨平台性。

JVM主要做了两件事：
1. 翻译
    - 将java字节码翻译成操作系统能够阅读的二进制
2. 内存管理
    - 完全由JVM管理内存，开发人员不需要关心内存。
    - 缺陷：一旦发生内存泄漏、内存溢出等问题，排查困难（需要深入JVM机制）

# JVM内存管理
## JVM运行时数据区
JVM运行时数据区包含五大部分：（城站笨方法。对）
1. 程序计数器（Program Counter Registry）
    - 是什么
        - 线程独占
        - 记录当前线程运行中的指令地址
    - 存在的意义
        - 为什么要记录呢？因为CPU调用线程的时间片是抢占式的，正在执行的线程可能会被挂起。
2. 栈（Java Stack）
    - 是什么：线程独占。每创建一个线程就会创建一个对应的栈。内部由多个栈帧组成。
    - 栈帧：每执行一个方法就创建一个栈帧。一个栈帧对应一个方法。由4个元素组成。
        - 局部变量表：存储八大基本数据类型值、对象引用地址
        - 操作数栈：用于装载局部变量表中的数据进行操作，并返回结果给局部变量表。
        - 动态链接：常量池中存储了方法的引用，方法A引用B，B引用C。栈帧中调用了方法A的引用，在运行过程中最终会指向C，这一个过程就是动态链接
        - 返回地址：方法执行结束之后返回的地址。一种是正常的返回，一种是抛出异常后的返回。
3. 本地方法栈（Native Method Stack）
    - 是什么：和Java栈的区别是使用了本地方法的栈。
4. 方法区（Method Area）
    - 是什么：存储类和对象的内存区域。主要存储类信息、类的字段信息、类的方法信息、final常量、静态变量。不会被GC回收。
5. 堆（Heap）
    - 是什么：存储对象的内存区域。具体的内存存储模型见JVM内存模型。

## JVM内存模型
| 新生代           | 老年代 | 永久代 |
| ---------------- | ------ | ------ |
| eden(8)          | 老年代 | 方法区 |
| from survivor(1) | 老年代 | 方法区 |
| to survivor(1)   | 老年代 | 方法区 |

堆中内存分成新生代和老年代。新创建的对象保存在新生代中的eden，之后将没有被回收的对象转移到survivor中。

# GC
## GC回收查找算法
有两种对象的GC查找算法
1. 引用计数法
2. 可行性分析

引用计数法就是查看某一个对象是否存在引用他的对象，如果存在，则不尽兴回收。但是如果存在循环引用的情况，就会失效，永远不会被回收。进而引出了可行性分析的算法，定义一个GC Root，只要没有被GC Root引用的对象都可以被GC回收。那么可以当作GCRoot呢？方法区的final常量、static静态变量、栈帧中引用的对象都可以被当作GCroot。因为方法区是永久区，不会被回收，那么就可以当作GCRoot，栈帧中的引用是运行时的引用，不可能把运行中的对象回收掉，因此也是可以被当作GCRoot的。

JVM使用了CAS同步保证多线程在堆里面开辟空间没有问题。

## 什么时候需要被GC回收
理想状态是：所有情况下都不要把对象放入老年代。因为老年代的MajorGC的回收效率比MinorGC效率低很多。

当Eden区域满的时候，就要触发一次GC回收，这次回收被称为MinorGC。这次MinorGC回收不再使用的对象，并会将Eden中幸存的对象（GC可达的）复制到其中一个Survivor中，清空原有的对象。      

当Eden区域第二次满了之后，触发了第二次MinorGC，这时就会将之前的Survivor区域中的还能够幸存下来的对象复制到另一个Survivor中，并清空原有的对象。这样就可以尽量的让对象的回收在新生代中进行。 

只有当Survivor满了，才会把对象放入到老年代。

## GC回收算法
1. 标记-清除算法
2. 复制回收算法
3. 标记-整理算法

垃圾回收器是回收算法的实现。