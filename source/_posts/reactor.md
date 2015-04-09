title: reactor代码浅析
date: 2014-08-27 18:44:12
updated: 2014-08-27 18:44:12
tags:
 - 异步
 - 并发
 - 开源
 - reactor
 - reactive streams
 - mampa
categories:

---
![](http://lyso.qiniudn.com/reactor.jpg)

## 简介

从[Reactive-Streams](http://lyso.me/2014/08/26/backpressure/) 的实现列表中看到了[Reactor](https://github.com/reactor/reactor/)， 但通读了其代码发现完全与reactor streams不沾边，而其做的事情与[MAMPA](http://lyso.me/2014/02/01/mampa/) 很接近，还是整理一下吧。

Reactor是一套在JVM上实现异步应用的框架，为Java、Groovy等JVM语言提供更便捷地搭建事件/数据驱动应用程序的底层抽象和接口，例如publish/consume事件。Reactor号称自己很快，在流行的硬件情况下，用其非阻塞Dispatcher吞吐能达到15,000,000的QPS。一个Reactor可以由多个不同的Dispatcher来实现，提交到Reactor的Task可以根据不同的配置而运行在Actor中的单个线程上/线程池中的一个线程/或LMAX Disruptor RingBuffer，如何配置可以根据业务类型来决定，比如阻塞IO操作用线程池，非阻塞快速计算用RingBuffer，等等。

## 用法

可以参见其[Usage-Guide wiki](https://github.com/reactor/reactor/wiki/Usage-Guide)，简单来说，

```
    // 1. 创建一个环境
    final Environment env = new Environment();

    // 2. 通过环境创建一个Reactor
    Reactor r = Reactors.reactor().env(env).get();
    
    // 3. 告诉Reactor如何处理事件（指定handler，即Consumer）
    reactor.on($("topic"), new Consumer<Event<Message>>() { ... });

    // 4. 向reactor中发送消息
    Message msg = msgService.nextMessage();
    reactor.notify("topic", Event.wrap(msg));
```

接下来按从整体到局部、从数据源头到尾的方式，简单分析其代码架构。

## Observable/Reactor

Reactor的接口定义为Observable，主要提供了三种接口。

### respondsToKey

查询是否这个key在Reactor中有对应的处理
```
    /**
     * Are there any {@link Registration}s with {@link Selector Selectors} that match the given {@code key}.
     *
     * @param key
     *         The key to be matched by {@link Selector Selectors}
     *
     * @return {@literal true} if there are any matching {@literal Registration}s, {@literal false} otherwise
     */
    boolean respondsToKey(Object key);
```

### on

将Consumer通过Selector注册到Reactor上，如果有消息到达Reactor并在Selector中match了，会发到这个Consumer。

```
    /**
     * Register a {@link reactor.function.Consumer} to be triggered when a notification matches the given {@link
     * Selector}.
     *
     * @param sel
     *         The {@literal Selector} to be used for matching
     * @param consumer
     *         The {@literal Consumer} to be triggered
     * @param <E>
     *         The type of the {@link reactor.event.Event}
     *
     * @return A {@link Registration} object that allows the caller to interact with the given mapping
     */
    <E extends Event<?>> Registration<Consumer<E>> on(Selector sel, Consumer<E> consumer);
```

### notify

notify接口有很多种，大同小异，相当于MAMPA中的tell，告诉Reactor这个事件可以进行处理了，处理完后调用参数中consumer的onComplete。

```
    /**
     * Notify this component that an {@link Event} is ready to be processed and {@link Consumer#accept accept} {@code
     * onComplete} after dispatching.
     *
     * @param key
     *         The key to be matched by {@link Selector Selectors}
     * @param ev
     *         The {@literal Event}
     * @param onComplete
     *         The callback {@link Consumer}
     * @param <E>
     *         The type of the {@link Event}
     *
     * @return {@literal this}
     */
    <E extends Event<?>> Observable notify(Object key, E ev, Consumer<E> onComplete);
```

### Reactor中的成员

一个Reactor中管理着如下几种成员对象。

```
    private final Dispatcher                             dispatcher;
    private final Registry<Consumer<? extends Event<?>>> consumerRegistry;
    private final EventRouter                            eventRouter;
    private final Consumer<Throwable>                    dispatchErrorHandler;
    private final Consumer<Throwable>                    uncaughtErrorHandler;
```

  * 当外部调用`on`接口时，会将Consumer的注册信息加入到`consumerRegistry`中去
  * 一个事件通过`notify`接口发送到Reactor时，Reactor通过`dispatcher`进行处理，从`comsuerRegistry`中找到响应的consumer进行处理
  * dispatcher进行分发时，需要`eventRouter`做路由
  * 在事件的分发过程中如果出现异常，会调用`dispatchErrorHandler`来处理
  * 在Reactor初始化时会调用`on`注册一个单独的Selector<Throwable>，使用`uncaughtErrorHandler`来处理

## Dispatcher

Dispatcher中主要是一个`dispatch`接口，负责将事件分发到对应的consumers上。

```
    /**
     * Instruct the {@code Dispatcher} to dispatch the {@code event} that has the given {@code key}. The {@link Consumer}s
     * that will receive the event are selected from the {@code consumerRegistry}, and the event is routed to them using
     * the {@code eventRouter}. In the event of an error during dispatching, the {@code errorConsumer} will be called. In
     * the event of successful dispatching, the {@code completionConsumer} will be called.
     *
     * @param key                The key associated with the event
     * @param event              The event
     * @param consumerRegistry   The registry from which consumer's are selected
     * @param errorConsumer      The consumer that is invoked if dispatch fails. May be {@code null}
     * @param eventRouter        Used to route the event to the selected consumers
     * @param completionConsumer The consumer that is driven if dispatch succeeds May be {@code null}
     * @param <E>                type of the event
     * @throws IllegalStateException If the {@code Dispatcher} is not {@link Dispatcher#alive() alive}
     */
    <E extends Event<?>> void dispatch(Object key,
                                                                         E event,
                                                                         Registry<Consumer<? extends Event<?>>> consumerRegistry,
                                                                         Consumer<Throwable> errorConsumer,
                                                                         EventRouter eventRouter,
                                                                         Consumer<E> completionConsumer);
```

在Reactor通过`notify`收到一个事件时，直接调用dispatcher的`dispatch`接口。在`dispatch`中，申请一个Task，然后提交执行：

```
    Task task;
    boolean isInContext = isInContext();
    if (isInContext) {
        task = allocateRecursiveTask();
    } else {
        task = allocateTask();
    }

    task.setKey(key)
        .setEvent(event)
        .setConsumerRegistry(consumerRegistry)
        .setErrorConsumer(errorConsumer)
        .setEventRouter(eventRouter)
        .setCompletionConsumer(completionConsumer);

    if (isInContext) {
        addToTailRecursionPile(task);
    } else {
        execute(task);
    }
```

至于提交执行的实现，视不同的不同的Dispatcher实现而异，Reactor中Dispatcher实现有很多种，如

  * ActorDispatcher
  * EventLoopDispatcher
  * RingBufferDispatcher
  * NettyEventLoopDispatcher
  * ThreadPoolExecutorDispatcher
  * WorkQueueDispatcher

比如RingBufferDispatcher的`execute`是直接将Task丢到实现申请好的RingBuffer的slot中：

```
    protected void execute(Task task) {
        ringBuffer.publish(((RingBufferTask) task).getSequenceId());
    }
```

其Task中的sequenceId是在`dispatch`中`allocateTask`时阻塞申请到的：

```
    protected Task allocateTask() {
        long seqId = ringBuffer.next();
        return ringBuffer.get(seqId).setSequenceId(seqId);
    }
```

然后RingBuffer的消费线程会将ringBuffer中的Task依次消费执行。

又如`NettyEventLoopDispatcher`则更简单，直接将Task提交到Excutor上：

```
    protected void execute(Task task) {
        eventLoop.execute(task);
    }
```

### Task

Task的执行是做了什么事情呢？调用EventRouter进行分发。

```
    protected static void route(Task task) {
        if (null == task.eventRouter) {
            return;
        }
        try {
            task.eventRouter.route(
                    task.key,
                    task.event,
                    (null != task.consumerRegistry ? task.consumerRegistry.select(task.key) : null),
                    task.completionConsumer,
                    task.errorConsumer
            );
        } finally {
            task.recycle();
        }
    }
```

## EventRouter

EventRouter进行分发的逻辑如下（以ConsumerFilterEventRouter为例），首先过滤出所有符合要求的Registration列表：

```
    if (null != consumers && !consumers.isEmpty()) {
        List<Registration<? extends Consumer<? extends Event<?>>>> regs = filter.filter(consumers, key);
```

然后对每一个Registration，调用其Selector的HeaderResolver将headers塞到event里，并调用`ConsumerInvoker`的`invoke`：

```
        int size = regs.size();
        // old-school for loop is much more efficient than using an iterator
        for (int i = 0; i < size; i++) {
            Registration<? extends Consumer<? extends Event<?>>> reg = regs.get(i);

            if (null == reg || reg.isCancelled() || reg.isPaused()) {
                continue;
            }
            try {
                if (null != reg.getSelector().getHeaderResolver()) {
                    event.getHeaders().setAll(reg.getSelector().getHeaderResolver().resolve(key));
                }
                consumerInvoker.invoke(reg.getObject(), Void.TYPE, event);
```

在出现问题时如果遇到CancelConsumerException则将Registration取消，对其他异常调用`errorConsumer`进行处理。

```
            } catch (CancelConsumerException cancel) {
                reg.cancel();
            } catch (Throwable t) {
                if (null != event.getErrorConsumer()) {
                    event.consumeError(t);
                } else if (null != errorConsumer) {
                    errorConsumer.accept(t);
                } else {
                    logger.error("Event routing failed for {}: {}", reg.getObject(), t.getMessage(), t);
                    if (RuntimeException.class.isInstance(t)) {
                        throw (RuntimeException) t;
                    } else {
                        throw new IllegalStateException(t);
                    }
                }
            } finally {
                if (reg.isCancelAfterUse()) {
                    reg.cancel();
                }
            }
        }
    }
```

完成转发后如果`completionConsumer`存在则执行其accept：

```
    if (null != completionConsumer) {
        try {
            consumerInvoker.invoke(completionConsumer, Void.TYPE, event);
        } catch (Exception e) {
            if (null != errorConsumer) {
                errorConsumer.accept(e);
            } else {
                logger.error("Completion Consumer {} failed: {}", completionConsumer, e.getMessage(), e);
            }
        }
    }
}
```

## Registry、Registration

一个Registry管理一个Reactor中所有的Regitration，可以认为它就是一个Registration的List，有接口`register`，`unregister`，`clear`：

```
    /**
     * Assign the given {@link reactor.event.selector.Selector} with the given object.
     *
     * @param sel The left-hand side of the {@literal Selector} comparison check.
     * @param obj The object to assign.
     * @return {@literal this}
     */
    <V extends T> Registration<V> register(Selector sel, V obj);

    /**
     * Remove any objects matching this {@code key}. This will unregister <b>all</b> objects matching the given
     * {@literal key}. There's no provision for removing only a specific object.
     *
     * @param key The key to be matched by the Selectors
     * @return {@literal true} if any objects were unassigned, {@literal false} otherwise.
     */
    boolean unregister(Object key);

    /**
     * Select {@link Registration}s whose {@link Selector} {@link Selector#matches(Object)} the given {@code key}.
     *
     * @param key The key for the Selectors to match
     * @return A {@link List} of {@link Registration}s whose {@link Selector} matches the given key.
     */
    List<Registration<? extends T>> select(Object key);

    /**
     * Clear the {@link Registry}, resetting its state and calling {@link Registration#cancel()} for any active {@link
     * Registration}.
     */
    void clear();

```

可以看到，`register`接口会返回`Registration`，Registration描述了一个`Selector`和一个`Consumer`的对应关系，有`getSelector`，`getObject`，`cancel`等接口，其中getObject就是获取Consumer：

```
    /**
     * The {@link reactor.event.selector.Selector} that was used when the registration was made.
     *
     * @return the registration's selector
     */
    Selector getSelector();

    /**
     * The object that was registered
     *
     * @return the registered object
     */
    T getObject();

    /**
     * Cancel this {@literal Registration} by removing it from its registry.
     *
     * @return {@literal this}
     */
    @Override
    Registration<T> cancel();
```

在Reactor中的`on`接口中就是执行了注册操作：

```
    public <E extends Event<?>> Registration<Consumer<E>> on(Selector selector, final Consumer<E> consumer) {
        Assert.notNull(selector, "Selector cannot be null.");
        Assert.notNull(consumer, "Consumer cannot be null.");
        Registration<Consumer<E>> reg = consumerRegistry.register(selector, consumer);
        return reg;
    }
```

而在Registry的`select`接口中，就是通过遍历其Registration的Selector的match来找到Registration列表来实现的：

```
    @Override
    public List<Registration<? extends T>> select(Object key) {
        // maybe pull Registrations from cache for this key
        List<Registration<? extends T>> selectedRegs = FastList.newList();

        // find Registrations based on Selector
        for (Registration<? extends T> reg : this) {
            if (reg.getSelector().matches(key)) {
                selectedRegs.add(reg);
            }
        }

        // nothing found, maybe invoke handler
        if (selectedRegs.isEmpty() && null != onNotFound) {
            onNotFound.accept(key);
        }

        // return
        return selectedRegs;
    }
```

## Selector

Selector的接口就不言自明了，需要提供`match`和`getHeaderResolver`：

```
    /**
     * Indicates whether this Selector matches the {@code key}.
     *
     * @param key The key to match
     *
     * @return {@code true} if there's a match, otherwise {@code false}.
     */
    boolean matches(Object key);

    /**
     * Return a component that can resolve headers from a key
     *
     * @return A {@link HeaderResolver} applicable to this {@link Selector} type.
     */
    HeaderResolver getHeaderResolver();
```

## Consumer

最后我们到了Consumer，看它的接口：

```
/**
 * Implementations accept a given value and perform work on the argument.
 *
 * @author Jon Brisbin
 * @author Stephane Maldini
 *
 * @param <T> the type of values to accept
 */
public interface Consumer<T> {

    /**
     * Execute the logic of the action, accepting the given parameter.
     *
     * @param t The parameter to pass to the consumer.
     */
    void accept(T t);

}
```

## Reactor vs. Mampa


