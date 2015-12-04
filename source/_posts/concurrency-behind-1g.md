title: 亿万并发的背后
date: 2015-07-01 18:44:12
updated: 2015-07-01 18:44:12
permalink: concurrency-behind-1g
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
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.001.jpeg)

## 送姜技术分享会
在组建送姜产品研发部的过程中，举办了几期送姜技术分享会（SJTC），其中第二期我主讲，总结之前在网易有道/小米做高并发服务积累的一些经验和认知，分享内容汇总如下。

![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.002.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.003.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.004.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.005.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.006.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.007.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.008.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.009.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.011.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.012.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.013.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.014.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.015.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.016.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.017.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.018.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.019.jpeg)
![concurrency](http://7jprdp.com1.z0.glb.clouddn.com/concurrency.arch.020.jpeg)
