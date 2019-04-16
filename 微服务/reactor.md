# 什么是Reactor？

Reactor是基于响应式数据流规范（Reactive Streams）的用于构建非阻塞式应用的响应式JVM库。

- 是什么：响应式JVM库（Reactive Library）
- 目的（作用）：构建非阻塞式应用（non-blocking application）
- 依赖：响应式数据流规范（Reactive Streams API）

在reactor中，当我们写了一个publisher chain，数据并没有写入到流中，相当于做了一个抽象的描述。

通过订阅，你可以将publisher绑定到subscriber上，从而触发整个链条的数据流。订阅者发布单一请求信号到上游传播，直到传回到源发布者。



## 响应式编程（Reactive Programming）

响应式编程可以说是观察者模式的扩展。迭代器基于拉，响应流基于推。

迭代器模式：由开发人员决定什么时候获取元素。

响应式编程：发布者在新的可用值出现时会通知订阅者，而这个推送就是响应式编程的关键。而且这个推送动作是声明式的而不是命令式的。程序员只需要关注业务逻辑计算，而不需要关注数据流的控制。

响应式编程除了推送值之外，还对异常处理和完成处理定义了一套良好的规范：一个发布者（Publisher）可以通过调用`onNext()`推送一个数值到对应的订阅者（Subscriber）。同时也可以发布一个错误（调用`onError()`）或者完成（调用`onComplete()`）的信号，从而中断整个序列。



## Reactor主要模块

- reactor-core
- reactor-test
- reactor-net
- reactor-bus
- reactor-stream



## 响应式数据流规范（Reactive Streams）

是什么：是一套响应式数据流规范。该规范定义了四个接口，1 个 TCK 和一些样例组成。

目的：提供一套同步/异步数据序列流式控制机制。

API定义的四个接口，即四个角色：

- `Processor`：处理器。处理器继承了发布者和订阅者。
- `Publisher`：数据流发布者（信号从0到N）。提供两个可选终端事件：错误和完成。
- `Subscriber`：数据流消费者
- `Subscription`：订阅信息。订阅的控制器 

```xml
<!-- 引入该响应式数据流规范依赖 -->
<dependency>
    <groupId>org.reactivestreams</groupId>
    <artifactId>reactive-streams</artifactId>
    <version>1.0.2</version>
</dependency>
```

### Publisher

数据流发布者（生产者）（信号从0到N）。提供两个可选终端事件：错误和完成。

```java
public interface Publisher<T> {
    /**
     * Request {@link Publisher} to start streaming data.
     */
    public void subscribe(Subscriber<? super T> s);
}
```

### Subscriber

数据流订阅者（消费者）（信号从0到N）。**消费者初始化过程中，会请求发布者当前需要订阅多少数据。**其他情况，通过接口回调与数据发布者交互：下一条（新消息）和状态。状态包括：完成/错误，可选。

订阅者有两种方式向发布者请求数据，如下所示：

- **无界的**：订阅者只需要调用 `Subscription#request(Long.MAX_VALUE)` 即可。
- **有界的**：订阅者保留数据引用，调用`request(long)`方法消费。
  - 通常订阅者在订阅时会请求一个初始数据集或者一个数据
  - 在 onNext 成功后(如 Commit，Flush 等… 之后)，请求更多数据
  - 建议请求数量呈线性，尽量避免请求叠加， 如每下一个信号请求 10 个数据

```java
public interface Subscriber<T> {
  // 订阅事件
  public void onSubscribe(Subscription s);
  // 数据到达事件
	public void onNext(T t);
  // 订阅异常
  public void onError(Throwable t);
  // 订阅完成事件
  public void onComplete();
}
```

### Subscription

初始化阶段将一个小追踪器传递给数据流消费者。它控制着我们准备好来消费多少数据，以及我们想要什么时候停止消费（取消）

```java
public interface Subscription {
  // 请求
  public void request(long n);
  // 取消
  public void cancel();
}
```

### Processor

同时作为发布者和订阅者的组件的标记。





# reactor核心（reactor-core）

reactor核心特点：

- 函数式功能。结合了JDK8之后的函数式接口
  - 比如：供应商Supplier、消费者Consumer、函数Function、谓语Predicate等等
- 两个类：Mono和Flux



## Mono和Flux

Mono是用于单元素的操作

Flux是用于N元素的操作

二者均实现了Publisher接口。作为发布者进行使用。在没有调用subscribe方法之前，所有的操作均只是声明。并为执行。等到执行subscribe方法之后，才会执行声明的方法体，实现非阻塞执行操作。

### 简单案例

```java
// 1. 简单订阅
@Test
public void simpleTest() {
  // 声明一个Flux。当订阅服务器绑定时生成10个值。
  Flux<Integer> flux = Flux.range(1, 10);
  // 底层会生成一个LambdaSubscriber对象。最终即调用订阅者
  flux.subscribe(consumer -> System.out.println(consumer));
}
```

###  异常中断处理

```java
// 异常中断处理
@Test
public void doErrorTest() {
    // 声明了一个publisher
    Flux<Integer> ints = Flux.range(1, 4).map(i -> {
        if (i <= 3) {
            return i;
        }
        throw new RuntimeException("Got to 4");
    });
    // 订阅
    ints.subscribe(i -> System.out.println(i), error -> System.err.println("Error: " + error));
}
```

### 完成处理

```java
// 4. 完成处理
@Test
public void doCompleteTest() {
    Flux<Integer> ints = Flux.range(1, 4);
    ints.subscribe(i -> System.out.println(i), error -> System.err.println("Error " + error), () -> System.out.println("Done"));
}
```



### 请求

```java
// 5. 请求
@Test
public void doRequestTest() {
    Flux<Integer> ints = Flux.range(1, 100);
    // 通过request设置请求只处理10个元素
    ints.subscribe(i -> System.out.println(i), error -> System.err.println("Error " + error), () -> System.out.println("Done"), sub -> sub.request(10));
}
```



### 取消

```java
// 6. 取消
@Test
public void doCancelTest() {
    Flux<Integer> ints = Flux.range(1, 11);
    ints.subscribe(i -> System.out.println(i), error -> System.err.println("Error " + error), () -> System.out.println("Done"), sub -> sub.cancel());
}
```



### subscribe

发起订阅，在底层会创建一个订阅者订阅当前发布者。在未订阅之前，发布者只是声明了操作，并不会执行操作。这就是所谓的同步非阻塞操作。



### map

map操作是纯粹的元素替换。将原有的每个元素替换成方法体中返回的数据

```java
Mono mono = Mono.just(1).flatMap(integer -> Mono.just(integer * 2));
mono.subscribe(list -> System.out.println(list));
// 输出结果：2

Flux.just(2, 4, 6, 8).map(integer -> integer/2).subscribe(obj -> System.out.print(obj + "\t"));
// 输出结果：1 2 3 4
```

### flatMap

flatMap操作也是元素替换。但不单纯是元素替换，需要添加新的发布者（Publisher），从而又会执行新的异步操作。

Flux中的flatMap的操作是无序的。不保证按照上游的数据顺序进行处理。

```java
Mono mono = Mono.just(1).flatMap(integer -> Mono.just(integer * 2));
mono.subscribe(list -> System.out.println(list));
// 输出结果：2

// Flux中的flatMap操作是不单纯是元素替换，需要添加新的发布者（Publisher），从而又会执行新的异步操作。
// Flux中的flatMap的操作是无序的。不保证按照上游的数据顺序进行处理
Flux flux = Flux.just(2, 4, 6, 8).flatMap(integer -> Flux.just(integer * 10 , integer * 1000, integer).delayElements(Duration.ofSeconds(1)));
flux.subscribe(list -> System.out.print(list + " "));
TimeUnit.SECONDS.sleep(20);
// 输出结果：20 2000 2 40 4000 4 60 6000 6 80 8000 8
// 输出结果：40 20 60 80 4000 8000 2000 6000 4 6 2 8
// 输出结果：40 20 60 80 4000 6000 8000 2000 4 8 6 2
// 输出结果：20 40 60 80 2000 4000 6000 8000 2 4 6 8
```



### concatMap

```java
// Flux中的concatMap操作和flatMap类似。但是严格按照上游数据顺序
Flux flux = Flux.just(2, 4, 6, 8).concatMap(integer -> Flux.just(integer * 10 , integer * 1000, integer).delayElements(Duration.ofSeconds(1)));
flux.subscribe(list -> System.out.print(list + " "));
TimeUnit.SECONDS.sleep(20);
// 输出结果：20 2000 2 40 4000 4 60 6000 6 80 8000 8
```



### range

声明生成一个范围的数据



### just

声明一个或多个数据



### then

当subscribe时，完成当前Mono，并开启一个新的Mono。

```java
public final <V> Mono<V> then(Mono<V> other);

Mono mono = Mono.defer(() -> {
  System.out.println(Thread.currentThread().getName() + "\t" + 1);
  return Mono.just(1);
}).then(Mono.just(2));
mono.subscribe(o -> System.out.println(Thread.currentThread().getName() + "\t o = " + o));
TimeUnit.SECONDS.sleep(2);
// 输出结果：
main	1
main	o = 2
```



### next

只保留Flux通道中的第一个元素进行下一步操作。

```java
Mono mono = Flux.just(2, 4, 6, 8).next()
  .flatMap(param -> Mono.just(param * 10));
mono.subscribe(param -> System.out.println(Thread.currentThread() + "\t" + param));
// 输出结果：Thread[main,5,main]	20
// 只会获取Flux数据流中的第一个数据。
```











