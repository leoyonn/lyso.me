title: back pressure及reactive streams研究
date: 2014-08-26 18:44:12
updated: 2014-08-26 18:44:12
tags:
 - 异步
 - 并发
 - 开源
 - Reactive Streams
 - back pressure
 - mampa
categories:

---
![back pressure-图片来自akka](http://lyso.qiniudn.com/backpressure.png)

## 引子
　　随着[Mampa](http://lyso.me/2014/02/01/mampa/) 越来越多的应用在线上系统中，我们在逐步优化同时，发现一个很难绕过去的问题。一个系统中可能有多个ActorGroup，每个ActorGroup中负责一中业务类型（或层次）。比如：

* FetchActor中负责fetch消息的接收、处理、发回
* RedisActor中负责消息中与redis请求相关的操作，发请求到redis-server并把server的返回结果解码后返回给FetchActor
* HBaseActor同RedisActor，负责处理HBase相关请求
* KakfaActor负责处理与Kafka相关的请求

　　如果当前HBase性能较差，RedisActor又没有挡住大量的请求，而FetchActor中过来的消息太快，根据[排队理论](http://lyso.me/2014/07/26/queue-threory-littles-law/) 中所述，对于HBaseActor这样一个子系统来说，如果消息到达速率 > 服务速率，这就不是一个稳定的系统，会导致Queue快速增长，直至内存爆掉或达到保护上限。到此上限后怎么处理更多的请求：

  - 丢弃
  - 将压力向上反馈到源头

　　第二种方法，就是我们所要系统考虑的back pressure。

## 什么是back pressure

　　这种情况一般成为“持续压力”（sustained load），如果系统设计时不慎重考虑过载的情况，可能造成的结果是完全无法服务。如果请求量超过了系统的处理能力，就是要做取舍决定的时候了：Something has to give。比较理想的降级服务方法是，让系统仍以正常的latency、以最高的吞吐运行，而高于这个吞吐的那部分请求就拒绝服务好了。

　　举一个过载例子。小米公司每周四都去奥体打羽毛球，8-10块场地，每块场地都>8个人，打起来都觉得等的时间长，汗都出不来，再加上2B的奥体服务态度，忒不爽了。如果采用报名制，每个场地报名6人，报名晚的就只能自己想办法了——比如自己花个钱定场地，那结果是绝得大多数都玩的爽。（当然不久后场地会换到西三旗伟士，没有2B管理员后打的会更爽）

　　什么是“Back Pressure”呢？再比如小米食堂里有500个座位，平均每人吃饭30min，如果同时有超过500个人到食堂，那就得有人等座，再加上饭菜不好吃肯定给差评了。如果可以即时在门口放一个牌子，显示已经进去了多少人（减去已经吃完出来的，就像很多车库的电子指示牌一样），发现食堂比较满的人就可以先不进去，去大桥上溜溜弯，去小米之家看会“万万想不到”，去咖啡厅坐会聊聊天…… 这样食堂里面正在吃饭的也觉得环境宽松，心情不错。根据Little's Law，食堂不能服务超过每小时1000人，如果12:00-12:30有600人来吃饭，那就有100人在咖啡厅或其它地方消遣。在线上系统亦是如此，超过系统处理能力的部分如果能及时反馈到咖啡厅、大桥上，就是实现了BackPressure。

## 什么是reactive streams

　　Reactive Streams 是一个在JVM上提供有非阻塞Back-Pressure能力的异步数据流的计划。

### 要解决的问题

　　处理实时数据流在异步系统里尤其需要小心，其中最主要的问题是在数据流的入口需要控制好不至于后续阶段中被冲垮。

　　Reactive streams的主要目标是在“异步边界”上控制数据流的流量，例如把数据传输给另一个线程或线程池时需要确保其不会buffer溢出。换句话说，Back Pressure 是整个模型中重要的环节以确保队列大小的可控（[队列：稳定的系统](http://lyso.me/2014/07/26/queue-threory-littles-law/)）。 如果Back Pressure是同步的，那整个系统的异步带来的好处就不存在了。

### First Draft Specification

Available immediately is a First Draft Specification covering:

* [Semantics](https://github.com/reactive-streams/reactive-streams/blob/v0.3/tck/src/main/resources/spec.md) —a specification document
* [API](https://github.com/reactive-streams/reactive-streams/tree/v0.3/spi/src/main/java/org/reactivestreams/api/) —Java interfaces for end users
* [SPI](https://github.com/reactive-streams/reactive-streams/tree/v0.3/spi/src/main/java/org/reactivestreams/spi/) —Java interfaces for implementations
* [TCK](https://github.com/reactive-streams/reactive-streams/tree/v0.3/tck/src/main/java/org/reactivestreams/tck/) —a test harness to validate implementations and guide implementors
All of the parts of the Draft Proposal is released under Creative Commons Zero (Public Domain).

### Implementations of the draft spec

* Akka Streams
  * See this Activator template introducing the Akka Project implementation in Scala; a Java version will follow shortly.
  * Please give Feedback on the issue tracker.
* Reactor Composable
  * [Reactor (1.1+)](http://github.com/reactor/reactor)
  * Current Implementation Draft is being explored for 1.1 and onwards, see Reactor Composable
* RxJava
  * Support being prototyped and explored for inclusion in RxJava 1.0
### Implementors

To get started implementing the draft specification, it is recommended to start by reading the README, then taking a look at the Specification then taking a look at the TCK. If you have an issue with any of the above, please take a look at closed issues and then open a new issue if it has not already been answered.


## 接口定义

　　Reactive Streams的接口分为两种，用户使用的接口和实现使用的接口。

　　用户实现的接口包括Producer、Consumer、Processor，实现使用的接口包括Publisher、Subscriber、Subscription。

### Producer

　　Producer是消息源，底层实现使用Publisher。Producer中有2个接口，

```
public interface Producer<T> {
    public Publisher<T> getPublisher();

    public void produceTo(Consumer<T> consumer);
}
```

* getPublisher：获取底层实现的Publisher
* produceTo：将制定的consumer连接到此producer上，在底层实现上需要让consumer去subscribe这个producer的publisher，之后producer产生的stream就能一直被这个consumer消费了，直到以下情况之一：
  * Stream里没有数据了
  * Producer抛异常了
  * Consumer取消接收更多消息

### Consumer

　　Consumer消费消息的stream，由Subscriber实现，只提供一个getSubscriber接口。

```
public interface Consumer<T> {
  public Subscriber<T> getSubscriber();
}
```

### Processor

　　Processor包含了Producer和Consumer的功能，consume <I>类型的消息，produce <O>类型的消息：

```
public interface Processor<I, O> extends Consumer<I>, Producer<O> {
}
```

### Publisher

　　Publisher是消息源，一个或多个Subscriber会连接到publisher上接收publisher产生的消息，通过Subscription的`requestMore`接口来决定是否生成更多消息。

```
public interface Publisher<T> {
  public void subscribe(Subscriber<T> subscriber);
}
```

### Subscriber

　　Subscriber从Publisher接收消息，通过Subscription的`requestMore`接口向publisher提交“需求”，只有当“需求”和publisher的消息都有的情况下才会接收到更多消息。

　　Subscriber被传送到Publisher进行subscribe时，如果producer接受此subscribe，会调用`onSubscribe`接口，如果拒绝会调用`onError`接口。

　　Publisher在条件满足（有需求且有更多的消息）的情况下调用`onNext`向Subscriber发送消息。

　　如果Publisher已经结束产生更多消息，会调用`onComplete`通知Subscriber。

　　如果生成消息的stream出了问题无法恢复，也会调用`onError`通知Sbuscriber。

```
public interface Subscriber<T> {
  public void onSubscribe(Subscription subscription);
  
  public void onNext(T element);
  
  public void onComplete();
  
  public void onError(Throwable cause);
}
```

### Subscription

　　Subscription描述一个Subscriber对一个Publisher的注册，注册成功后才可以通过这个Subscription的`requestMore`接口向Producer索取更多消息。

　　可以通过`cancel`接口取消掉这个subscription，取消时即使Publisher还有更多消息，也会停止向对应的Subscriber发送。

```
public interface Subscription {
  public void cancel();
  
  public void requestMore(int elements);
}
```

## Mampa里怎么做

TO BE ADDED...