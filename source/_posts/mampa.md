title: 小米异步消息处理架构：mampa
date: 2014-02-01 18:44:12
updated: 2014-02-10 18:44:12
permalink: mampa
tags:
 - 异步
 - 并发
 - actor模型
 - 有限状态机
 - FSM
 - erlang
 - java线程模型
 - java内存模型
 - 开源
categories:

---
![mampa](http://lyso.qiniudn.com/mampa.jpg)

## 什么是mampa
*MAMPA*的全称是：*xiaoMi Asynchronous Message Processing Architecture*，是我在小米消息组时用java开发的一套异步并发框架。

## 为什么做mampa
　　问各位一个问题：如何写并发程序？大家脑海中会浮现一堆关键词，synchronized, ReentrantReadWriteLock, Condition, await, notify, ThreadLocal, Lock, LinkedBlockingQueue, ConcurrentHashMap, volatile …… 也能想到一些其他的语言，如Erlang、Golang等。我们需要考虑到对象是需要对象和类是不是线程安全的，是不是需要在多个线程上运行，是不是要加锁，会不会造成死锁，会不会有饥饿，要不要为性能失掉一定的正确性，等等。在这种情况下，新手上路很难写出正确高效漂亮兼具的代码，高手和大牛的经验不具有可复制性可可扩展性。

　　小米消息系统的架构和业务逻辑代码大多用Erlang编写，这里另外一个越来越突出的问题是，大规模Erlang程序学习曲线太高，新手很难在短时间内参与到线上系统的开发和问题调查中去，这样会导致个别老员工疲于奔命……

　　一开始，老员工想做出这套erlang程序的java版。但后来Team Lead找到我，说我们应该着眼与更长远的目标，做一套通用的框架，让新同事写业务逻辑时就像连连线，不用考虑多线程问题。于是我开始研究Erlang中的gem_fsm，研究akka中的actor，在组里其他同事的协助和宝贵建议下，mampa出炉并一步一步走向成熟，逐步接管小米云平台消息系统各个业务中的各种模块。

## 设计核心
　　我们的目标是，*设计一个高吞吐高性能的异步消息处理架构，并对开发者将并发问题隐藏起来*。

### 从头开始
　　什么样的并发程序/机制是高效的？加锁？共享内存？信号量？

　　要回答这个问题，我们不妨想想，自然界的并发单位是什么？我们每个人都是一个并发单位，我们都在做着自己的事情，思考着自己的问题——哪里有什么共享内存？人与人之间交流，用说话，用短信，电子邮件，为什么需要在你大脑里的一块区域加锁，然后把要告诉你的信息放进去，然后再释放锁？哪有什么锁的概念？我想问题时到我自己大脑检索信息，难道也要锁不成？

　　我接触到这个概念是恍然大悟“是啊，是这样的”，不知道你怎么想。那在写程序时，为什么不能这样呢。这是MAMPA的设计核心思路——无锁。

### Actor：并发单元
　　在MAMPA中，一个并发单元就是Actor。每个Actor锁定在一个线程上：这个Actor中的所有数据只有这个线程能处理、这个线程也只为这个Actor服务。

　　有了Actor，怎么与之通信呢？[Golang](http://golang.org/doc/effective_go.html) 有一个我很喜欢的设计理念，

> Do not communicate by sharing memory; instead, share memory by communicating.

　　在Golang里是通过channel传输消息的，那我们也需要给Actor做一个专用的channel，叫做mailbox。任何Actor想与其他Actor通信，就需要把消息发到他们的mailbox中，也就是说mailbox可以任意多个线程写，只有一个线程读——多线程的竞争问题只在此处体现，除此之外，全是单线程的天下，即除了mailbox的其它任意数据只有一个线程读写。

　　那如何高效实现这样的mailbox呢？当然有一种简单的实现方案是LinkedBlockingQueue。这个场景我们的第一反应是[Disruptor](https://github.com/LMAX-Exchange/disruptor) ，一个高效的无锁环形队列。对Disruptor的介绍参见我[另一篇文章](http://lyso.me/2013/09/01/disruptor/) 。


## 应用案例
## mampa性能好在哪


