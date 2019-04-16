---
title:JavaSE
---



## 概述

这个项目是关于JavaSE的。

## 封装
> 概念：将数据和访问数据的方法绑定在一个类中。通过统一的入口进行访问控制。
1. 基本数据类型和封装类
    - 8种基本数据类型：byte、short、int、long；double、float；char；boolean
    - 基本数据类型不涉及到对象的构造和回收，存在内存栈中。速度快。
    - 封装类是引用类型。值存在内存堆中，引用存在栈中。便于java操作。
2. 访问权限修饰符public、protected、private以及不写的区别

备注：自动装箱是将基本数据类型自动转换成封装类，无需程序员编写相关转换代码。

| 作用域    | 自己（当前类） | 朋友（同一个包类） | 子女（继承子类） | 陌生人（其他包的类） |
| --------- | -------------- | ------------------ | ---------------- | -------------------- |
| public    | ok             | ok                 | ok               | ok                   |
| protected | ok             | ok                 | ok               | no                   |
| 不写      | ok             | ok                 | no               | no                   |
| private   | ok             | no                 | no               | no                   |

## 继承
> 概念：在已有类的基础上创建新的类，新的类继承了已有类的信息。实现了代码的重用，软件系统的延续性。
1. 抽象类和接口的区别
    - 不同点
        - 1. 抽象类可以定义成员变量，接口不行；
        - 2. 抽象类可以有抽象方法和具体方法，接口只定义抽象方法。
        - 3. 抽象类可以定义任意访问权限的方法，接口的方法都是public的。
        - 4. 有抽象方法的类必须定义为抽象类，抽象类不一定包含抽象方法；
        - 5. 抽象类可以包含静态方法，接口不行；
        - 6. 一个类只能继承一个抽象类，可以实现多个接口；
        - 7. 抽象类可以定义构造方法，接口不行；
    - 相同点
        - 1. 不能够实例化
        - 2. 可以将抽象类和接口作为引用类型
        - 3. 一个类实现或者继承了接口和抽象类，必须对抽象方法进行实现，否则只能定义为抽象类。
2. 是否可以继承基本数据类型
    - 无法继承基本数据类型，因为不是对象
3. 是否可以继承String或者封装类？
    - 无法继承。因为都是final类

## 多态
### 实现机制
1. 父类引用指向子类对象。程序调用的方法是内存中实例对象的方法，而不是引用变量定义的方法。
2. 重写和重载。都是实现多态的方法。
    - 重载：编译时的多态性
        - 在父类、子类和同类中；方法名必须相同；参数类型或者个数或者顺序不同；
        - 返回类型不同不构成重载。因为方法调用时不指定返回类型。编译器无法知晓调用哪个返回类型的方法；
        - 可以抛出不同的异常，有不同的修饰符。
    - 重写：运行时的多态性
        - 方法名、参数、返回类型必须和父类保持一致；
        - 构造方法、final修饰方法、static修饰方法，不能重写；static方法可以在子类中重新声明。
        - 访问权限不能比父类低；

## 数据类型
### 基本数据类型
> 八大基本数据类型，存放在内存栈中，速度快。对应有封装类，引用对象存于栈，实例存于内存堆。对象的操作是通过封装类进行的。
> 所以会存在自动装箱的操作。（将基础数据类型转换成封装类）

##### 八大基本数据类型
1个字符，2个字节；1个字节，8位。
1. boolean
    - bool型，1位
    - 默认false
2. char
    - 字符型，2个字节
    - 可以保存一个中文字。中文字占2个字节。
3. byte
    - 字节类型，8位
    - byte变量占用的空间是int的四分之一
    - 以二进制表示的整数
    - 默认为0
4. short
    - 短整型，16位
    - 占用空间是int的二分之一
    - 默认0
5. int
    - 整型，32位
    - 默认0
6. long
    - 长整型，64位
    - 占用空间是int的两倍。一般用于大数据量。
    - 默认0L
7. float
    - 浮点型，32位
    - 默认0F
8. double
    - 浮点型，64位
    - 浮点数的默认类型是double
    - 默认0D
### 字符类型
1. String
    - String是只读字符串，无法修改原本对象，只能需改引用。
    - 因此在需要多次修改String内容的时候，相当于在内存中又加了对象，会造成内存资源消耗过大，性能低下，需要用StringBuffer来直接修改值。
2. StringBuffer
    - 直接修改字符串对象。
3. StringBuilder
    - 用法和StringBuffer一样。
    - 方法没有用synchronized修饰，因此只适用于单线程环境。效率相对高

## 语法
### 修饰符
1. 访问修饰符
    - public：所有对象都能访问
    - protected：自己和朋友以及子女可以访问
    - default：自己和朋友可以访问
    - private：只能自己访问
2. final
    - final变量：变量只能初始化一次
    - final方法：方法不能被重写
    - final类：类不能被继承
3. static
    - 静态变量：属于类的变量，无论初始化几次都只又一份内存堆。
    - 静态方法：属于类的方法，无论初始化几次都只又一份内存堆。不能调用非静态变量和非静态方法。
4. synchronized
    - 同步方法：用于声明该方法只能一次被一个线程使用（单机模式）
    - 同步代码块：用于声明该代码块只能一次被一个线程使用（单机模式）
5. abstract
    - 抽象类：不能被final修饰
    - 抽象方法：不能被final和static修饰
6. volatile
    - 被volatile修饰的变量被线程访问时总是从共享内存中取值，并且在线程修改该数据的时候，总是修改共享内存的数据。
7. transient
    - 被transient修饰的变量不会被序列化
8. native
    - 被native修饰的方法是通过本地C++实现的方法

## 反射
> 通过反射机制，可以访问对象的属性、方法、注解等信息。

```
// Java提供的反射机制的类
java.lang.Class
java.lang.reflect.Constructor
java.lang.reflect.Method
java.lang.reflect.Field
java.lang.reflect.Modifire
```
##### 实现机制
```
// 方式一：Class.forName
Class c = Class.forName("org.raccoon.reflex.Apple");

// 方式二：
Class c = Apple.class;

// 方式三：
Apple apple = new Apple();
Class c = apple.getClass();

Method[] methods = c.getMethods();
Field[] fields = c.getFields();
Annotation[] annotations = c.getAnnotations();
```

## 动态代理
> 概念：动态代理和静态代理的区别
1. 静态代理和动态代理的区别
    - 静态代理知道要代理的是什么；动态代理不知道，运行时才知道；
    - 静态代理是代理一个类；动态代理是代理一个接口下的多个实现类；
2. 动态代理的实现方式
    - 接口代理：通过JDK的InvocationHandler的invoke方法，业务类必须实现接口，通过Proxy类的newProxyInstance方法获取代理对象。
    - 类代理：CGLIB。通过派生的子类，来实现代理。
3. 使用代理的好处
    - 添加日志、异常处理、权限检查、缓存功能等

使用静态代理，就需要在每一个调用原本类方法的地方加上代理类。这样的工作在开发效率上过于繁琐。因此引入了动态代理。

Spring AOP就是基于接口动态代理实现的。

## IO
> 1. 两个对应一个桥梁。两个对应分别是输入输出流和字节字符流，一个桥梁是将字节流转换成字符流的桥梁，对应的是InputStreamReader、OutputStreamWriter。      
> 2. 流里面又可以做细分：1. 介质流，主要指一些基本的流，主要从具体的介质（文件、内存缓存区（StringBuffer数组、byte数组、Char数组））上读取数据；2. 装饰流，主要对其包装的类进行特定的处理，如：缓存。
> 3. IO中涉及的设计模式：装饰器模式；

##### 输入字节流
0. 基类
     - InputStream是所有的输入字节流的父类。是抽象类。
     - 基本方法
         - abstract int read() ：读取一个字节数据，并返回读到的数据，如果返回-1，表示读到了输入流的末尾；
         - int read(byte[] b) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾；
         - int read(byte[] b, int off, int len) ：将数据读入一个字节数组，同时返回实际读取的字节数。如果返回-1，表示读到了输入流的末尾。off指定在数组b中存放数据的起始偏移位置；len指定读取的最大字节数。
         - void close() ：关闭输入流，释放和这个流相关的系统资源。
         - ...
     - 流结束的判断：方法read()的返回值为-1时；
1. 基本介质流
    - ByteArrayInputStream：将内存中的Byte数组适配为一个InputStream对象；
    - FileInputStream：最基本的文件输入流。主要用于从文件中读取信息。
2. PipedInputStream：在多线程程序中作为数据源，同样会使用其它装饰器提供额外的功能。
3. 装饰流
    - ObjectInputStream
    - FilterInputStream：给其它被装饰对象提供额外功能的抽象类
        - DataInputStream：一般和DataOutputStream配对使用,完成基本数据类型的读写。
        - BufferedInputStream：使用InputStream的方法读取，只是背后多一个缓存的功能。设计模式中透明装饰器的应用。
        - LineNumberInputStream：只是增加一个行号。可以象使用其它InputStream一样使用。
        - PushbackInputStream：一般仅仅会在设计compiler的scanner 时会用到这个类。在我们的java语言的编译器中使用它。很多程序员可能一辈子都不需要。
4. 流中的工具
    - SequenceInputStream：将两个或者多个输入流当成一个输入流依次读取
##### 输出字节流
0. 基类
     - OutputStream是所有输出字节流的父类。是抽象类。
     - 基本方法
         - abstract void write(int?b)：往输出流中写入一个字节。
         - void write(byte[]?b) ：往输出流中写入数组b中的所有字节。
         - void write(byte[]?b, int?off, int?len) ：往输出流中写入数组b中从偏移量off开始的len个字节的数据。
         - void flush() ：刷新输出流，强制缓冲区中的输出字节被写出。
         - void close() ：关闭输出流，释放和这个流相关的系统资源。
1. 基本介质流
    - ByteArrayOutputStream：一般将其和FilterOutputStream套接得到额外的功能。建议首先和BufferedOutputStream套接实现缓冲功能。通过toByteArray方法可以得到流中的数据。（不透明装饰器的用法）
    - FileOutputStream：一般将其和FilterOutputStream套接得到额外的功能。
2. PipedOutputStream
3. 装饰流
    - ObjectOutputStream
    - FilterOutputStream
        - DataOutputStream
        - BufferedOutputStream
4. 流中的工具
    - PrintStream：System.out和System.out就是PrintStream的实例！

##### 输入字符流
0. 基类
     - Reader是所有输入字符流的父类，是抽象类。
     - 基本方法
         - public int read() throws IOException; //读取一个字符，返回值为读取的字符 
         - public int read(char cbuf[]) throws IOException; // 读取一系列字符到数组cbuf[]中，返回值为实际读取的字符的数量 
         - public abstract int read(char cbuf[],int off,int len) throws IOException; // 读取len个字符，从数组cbuf[]的下标off处开始存放，返回值为实际读取的字符数量，该方法必须由子类实现 
1. 基本介质流
    - CharArrayReader
    - StringReader
2. PipedReader
3. 装饰流
    - BufferedReader
    - FilterReader
        - PushbackReader

##### 输出字符流
0. 基类
     - Writer是所有输出字符的父类，是抽象类。
     - 基本方法
         - public void write(int c) throws IOException； //将整型值c的低16位写入输出流 
         - public void write(char cbuf[]) throws IOException； //将字符数组cbuf[]写入输出流 
         - public abstract void write(char cbuf[],int off,int len) throws IOException； //将字符数组cbuf[]中的从索引为off的位置处开始的len个字符写入输出流 
         - public void write(String str) throws IOException； //将字符串str中的字符写入输出流 
         - public void write(String str,int off,int len) throws IOException； //将字符串str 中从索引off开始处的len个字符写入输出流 
1. 基本介质流
    - CharArrayWriter
    - StringWriter
2. PipedWriter
3. 装饰流
    - BufferedWriter
4. 流中的工具
    - PrintWriter：和PrintStream极其类似，功能和使用也非常相似

##### 桥梁
1. InputStreamReader
2. OutputStreamWriter
    - FileReader

##### 字节流和字符流的选择
字节流可以处理所有类型数 据，如:图片，MP3，AVI 视频文件，而字符流只能处理字符数据。只要是处理纯文本数据，就要优先考虑使用字符流，除此之外都用字节流。

##### 关于对象序列化
对象序列化就是将对象流化。便于网络传输。实现方式：类实现Serialized接口。通过FileInputStream转化对象。

[参考文献](https://blog.csdn.net/baobeisimple/article/details/1713797)

## 异常
1. 案例
```
public static void main(String[] args) {
    System.out.println(returnResult()); // out --> 2
}

private static int returnResult() {
    try {
        int a = 1/0;
    } catch (Exception e) {
        e.printStackTrace();
        return 1;
    } finally {
        return 2;
    }
}
```
2. 异常的种类
    - 运行时异常。常见的运行时异常有：
        - 数组越界java.lang.IndexOutOfBoundsException
        - 空指针
        - java.lang.IllegalArgumentException 方法传递参数错误。
        - java.lang.ClassCastException 数据类型转换异常。
        - java.lang.NumberFormatException 字符串转换为数字异常;出现原因:字符型数据中包含非数字型字符。
        - java.lang.ClassNotFoundException 指定的类找不到;出现原因:类的名称和路径加载错误;通常都是程序试图通过字符串来加载某个类时可能引发异常
        - java.lang.NoClassDefFoundException 未找到类定义错误
        - SQLException SQL 异常，常见于操作数据库时的 SQL 语句错误
        - java.lang.NoSuchMethodException 方法不存在异常。
    - 编译时异常
3. throw和throws
    - throw用于方法体内部抛出异常
    - throws用于声明某个方法会抛出哪种异常，并将异常处理交由上层处理

## 线程和并发
1. Runnable、Callable、Future
2. Thread
3. Executor框架：将任务的提交和执行区分开来
    - Executors：用于创建线程池
    - ExecutorService：Executors创建线程池的返回对象，用于提交执行任务
        - submit方法：有返回值Future
        - execute方法：无返回值
    - 线程池
        - 创建固定线程池，每提交一个任务就创建一个线程，直到达到线程池的最大数量。public static ExecutorService newFixedThreadPool();
        - 单线程Executor，创建单个工作者线程执行任务。任务在队列中是串行执行。public static ExecutorService newSingleThreadExecutor();
        - 创建一个可缓存的线程池，如果线程池的当前规模超过了处理需求，就会回收空闲线程，当需求增加时，就添加新的线程，线程池的规模不存在任何限制 public static ExecutorService newCachedThreadPool();
        - 创建固定线程池，以延迟或者定时的方式执行任务。public static ScheduledExecutorService newScheduledThreadPool(int nThreads);
    - 工作队列：保存了等待执行的任务列表
        - BlockingQueue
            - 阻塞方法：put/take
            - 非阻塞方法：offer/poll
    - 工作者线程：从工作队列中获取一个任务，执行这个任务，然后返回线程池等待下一个任务。
4. 安全性问题
    - 竞态条件：由于多线程访问共享数据进行非原子操作导致错误结果的安全问题。
5. 活跃性问题
    - 阻塞：由对同一共享数据的访问等待上一个线程释放资源所造成的活跃性问题。
    - 死锁：线程A占用了线程B所需要的资源1，线程B占用了线程A所需要的资源2，两者同时等待对方释放资源，因此陷入了无限等待，即死锁。
    - 饥饿：高优先级线程吞噬了所有低优先级线程时间片，使得低优先级线程一直等待进入同步代码块。
    - 活锁：线程一直处于运行中，但做的都是无用功，导致任务无法进行。例如"重试机制"，当线程失败之后，就会重试，如果一直失败，就无法继续进行任务。
6. 性能问题
    - CPU分配给各个线程的时间是时间片。在线程执行完成之后，CPU进行线程切换所需要花费的时间与CPU资源会产生性能问题。

各个问题的解决方案
> 1. 安全性问题：单机模式，添加synchronized锁；集群模式，添加分布式锁，比如redis分布式锁，保证操作的原子性。   
> 2. 活跃性问题：设置合理的线程优先级。  
>   在无法完全解决这些问题的情况下，解决的优先级是：安全性问题》活跃性问题》性能问题

## 设计模式
> java中的设计模式总共23种，没必要每种都掌握。下列是设计模式列表。针对需要掌握的设计模式我单独拎出来：     
> 创建型设计模式：单例模式、工厂方法模式、抽象工厂模式；建造者模式、原型模式     
> 结构型设计模式：代理模式、适配器模式、装饰器模式；外观模式、桥接模式、组合模式、享元模式      
> 行为型设计模式：策略模式、模板方法模式、观察者模式、中介者模式；迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、解释器模式。      
##### 1. 工厂方法模式
解决的问题：主要解决接口选择的问题。

##### 2. 抽象工厂模式
解决的问题：主要解决接口选择的问题。
和工厂方法模式的区别：抽象工厂模式是产生工厂的工厂，是个超级工厂。在工厂模式的基础上添加了一个抽象类的中间层。

##### 3. 单例模式
解决的问题：用于解决全局类的频繁创建和销毁的问题。

##### 4. 代理模式
目的：提供一个代理，来控制对这个对象的访问。
解决的问题：解决直接访问对象存在的问题。比如：要访问的对象在远程的机器上，直接访问该对象存在一定的问题和麻烦。
主要使用场景：1. 远程代理；2. 缓存。等等
和装饰器模式的区别：装饰器模式侧重于增强功能；代理模式侧重于对对象加以控制。

##### 5. 装饰器模式
目的：扩展一个类的功能
解决的问题：一般扩展一个类的功能需要继承，单就增强功能而言，装饰器模式比继承要灵活。

##### 6. 适配器模式
解决的问题：解决"现存的对象"不支持新的接口，但又要兼容这些"现存的对象"，因此需要适配器模式。

##### 7. 策略模式
解决的问题：解决在多种相似算法情况下，if...else语句可维护性太差的问题。

##### 8. 模板方法模式
解决的问题：一些方法通用，却在每一个子类都重新写了这一方法的代码过于冗余的问题。

##### 9. 观察者模式
解决的问题：一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。

##### 10. 中介者模式
解决的问题：对象与对象之间存在大量的关联关系，这样势必会导致系统的结构变得很复杂，同时若一个对象发生改变，我们也需要跟踪与之相关联的对象，同时做出相应的处理。

## JVM和类加载器和垃圾回收

### 容器和范型
1. Map
2. List
3. Set
4. HashMap
5. HashTable
6. ArrayList
7. HashSet
8. LinkedList
9. Collection
10. ConcurrentHashMap

## 表达式
##### 位移表达式
1. 左移表达式：数字*2的n次方
    - 1 << 3 = 8
2. 右移表达式：数字/2的n次方
    - 8 >> 2 = 2
3. 与：二进制表达式。全1出1。
    - 12（十进制）-》1100（二进制）
    - 4（十进制）-》100（二进制）
    - 12 & 4 = 1100 & 0100 = 0100 = 4
4. 或：二进制表达式。有1出1
    - 12（十进制）-》1100（二进制）
    - 4（十进制）-》100（二进制）
    - 12 | 4 = 1100 | 0100 = 1100 = 12
5. 非：二进制表达式
6. 异或：不同为1，相同为0
    - 12（十进制）-》1100（二进制）
    - 4（十进制）-》100（二进制）
    - 12 ^ 4 = 1100 ^ 0100 = 1000 = 8
7. 同或：相同为1，不同为0

## 语法
java中的goto语法。JDK中的多线程代码中运用到了该语法（ThreadPoolExcutor.addWorker()）。使用方式是：
1. 定义标签名称。比如：retry:
2. 在标签下面，写for循环的代码。标签和循环代码之间不允许添加其他代码。
3. 在循环代码中，在需要跳转的地方添加跳转代码 break:retry; 或者 continue:retry;

##### continue;和break;和break:retry;和continue:retry;的区别
1. continue;
    - 跳出本次的循环，进入到下一次循环；
2. break;
    - 跳出整个循环；
3. continue:retry;
    - 在自定义标签retry指定的循环处，跳出本次循环，进入到下一次循环；
4. break:retry;
    - 跳出由标签retry指定的循环；