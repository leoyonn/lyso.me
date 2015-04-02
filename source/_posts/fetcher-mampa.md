title: mampa应用之——小米推送Fetcher服务性能调校
date: 2014-08-05 21:22:11
tags: 
 - 内存泄露
 - mampa
 - 异步
 - 并发
 - 小米推送系统
categories: 
---
![](http://blog.xiaomi.com/wp-content/uploads/2014/03/mipush.jpg)

### 背景
小米消息推送系统在前不久开始使用[MAMPA](http://lyso.me/2014/02/01/mampa/) 进行异步化改造，改造的过程中，发现对hbase的读写是关键路径，而这一步骤是同步操作，与mampa的异步化框架是相悖的，可优化空间并不大，调试很久后性能甚至尚未达到原同步版：要么线程数开少了竞争不过其它的非异步化实例，要么线程数开太多导致load太高，性能更差。

随后我们分析到hbase性能方面的问题后，参考[Scaling memcache at Facebook](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf&sa=U&ei=gWJjU97pOeqxsQSDkYDAAg&ved=0CBsQFjAA&usg=AFQjCNGMeuWne9ywncbgux_XiZW6lQWHNw) [PPT](http://qcontokyo.com/pdf/qcon_MarcKwiatkowski.pdf) 设计了一套leased-cache，底层使用[redis-mampa](http://lyso.me/2014/03/10/redis-mampa/) （使用netty的异步化pipiline redis客户端）。

### 初步试验
#### 实验环境：
* hostA 作为压力测试客户端；
* hostB 作为fetcher业务服务端；
* hostC 作为redis、rabbitmq；
* hostD 作为mysql；
* Hbase访问采用mock数据并sleep一定时常方式；

#### 实验环境：
* HBase请求 sleep100ms左右；
* Fetcher主逻辑线程6个、redis线程实验1/2/4/8个、hbase-mampa线程64个；

#### 实验数据：
* 初始状态，cache里没有任何数据，请求全落到hbase：
  * qps = 600，latency.99 =800ms
* 以qps = 600将200w数据灌到redis中，模拟cache全命中情况：
  * qps = 10000, latency.99 = 800ms，redis的latency.average在30ms左右
  * 最大qps可以达到12000，此时查看数据生成导致mysql访问总是存在（mysql在主业务线程里访问，latency=0.5ms，6个线程qps峰值则为12000，后续可以通过mock 数据不需要访问mysql的情况来测试）

这里首先得到初步印象，latency的.99在800ms时，qps能达到10000，但没有获取到redis的latency.99以及这800ms的分布情况，而且有一个奇怪的现象是改变redis线程个数对结果影响不是太大，跟单独测试redis-mampa的性能情况不符。接下来进行更细致的实验和调校。

### 实验调校
#### 步骤1：qps=5000
```
main.thread=6
redis.thread=4
qps=5000
```

```
wait~before~process.HIST-75-percentile: 86.0
wait~before~process.HIST-95-percentile: 105.0
wait~before~process.HIST-99-percentile: 118.71000000000004
wait~before~process.HIST-999-percentile: 123.971

Request.lifetime.HIST-75-percentile: 73.0
Request.lifetime.HIST-95-percentile: 115.0
Request.lifetime.HIST-99-percentile: 142.71000000000004
Request.lifetime.HIST-999-percentile: 191.942
```

消息在系统中处理的总时间.99为142.71ms，latency是可以接受的，先看下这些时间的主要分布，之后增加qps后再放大地看是否存在问题：

```
mampa~event~inqueue~time.HIST-75-percentile: 2.0
mampa~event~inqueue~time.HIST-95-percentile: 14.0
mampa~event~inqueue~time.HIST-99-percentile: 46.710000000000036
mampa~event~inqueue~time.HIST-999-percentile: 59.971000000000004

redis~mampa~from~send.HIST-75-percentile: 22.0
redis~mampa~from~send.HIST-95-percentile: 57.0
redis~mampa~from~send.HIST-99-percentile: 64.71000000000004
redis~mampa~from~send.HIST-999-percentile: 91.71000000000004

(from tell是from send的超集，包括在actor的mailbox的时间）
redis~mampa~from~tell.HIST-75-percentile: 37.0
redis~mampa~from~tell.HIST-95-percentile: 65.0
redis~mampa~from~tell.HIST-99-percentile: 70.0
redis~mampa~from~tell.HIST-999-percentile: 96.85500000000002
```

#### 步骤2：qps=8000
```
main.thread=6
redis.thread=4
qps=8000
```

```
wait~before~process.HIST-75-percentile: 111.0
wait~before~process.HIST-95-percentile: 142.0
wait~before~process.HIST-99-percentile: 160.42000000000007
wait~before~process.HIST-999-percentile: 164.971

Request.lifetime.HIST-75-percentile: 118.0
Request.lifetime.HIST-95-percentile: 173.0
Request.lifetime.HIST-99-percentile: 228.0
Request.lifetime.HIST-999-percentile: 295.1880000000001
```

消息在系统中处理的总时间.99为228ms，主要分布如下：

```
mampa~event~inqueue~time.HIST-75-percentile: 4.0
mampa~event~inqueue~time.HIST-95-percentile: 26.549999999999955
mampa~event~inqueue~time.HIST-99-percentile: 74.97000000000025
mampa~event~inqueue~time.HIST-999-percentile: 99.0

redis~mampa~from~send.HIST-75-percentile: 49.0
redis~mampa~from~send.HIST-95-percentile: 165.0
redis~mampa~from~send.HIST-99-percentile: 196.71000000000004
redis~mampa~from~send.HIST-999-percentile: 207.0

redis~mampa~from~tell.HIST-75-percentile: 81.0
redis~mampa~from~tell.HIST-95-percentile: 174.54999999999995
redis~mampa~from~tell.HIST-99-percentile: 204.42000000000007
redis~mampa~from~tell.HIST-999-percentile: 210.0
```

#### 步骤3：qps=10000
```
main.thread=6
redis.thread=4
qps = 10000
```

```
wait~before~process.HIST-75-percentile: 131.0
wait~before~process.HIST-95-percentile: 227.0
wait~before~process.HIST-99-percentile: 463.5500000000002
wait~before~process.HIST-999-percentile: 499.884
Request.lifetime.HIST-75-percentile: 2451.5
Request.lifetime.HIST-95-percentile: 3307.0
Request.lifetime.HIST-99-percentile: 3501.42
Request.lifetime.HIST-999-percentile: 3581.980000000001
```

消息在系统中处理的总时间.99飙升到3.5s，比之前初步实验时的情况还差，主要分布如下：

```
mampa~event~inqueue~time.HIST-75-percentile: 43.0
mampa~event~inqueue~time.HIST-95-percentile: 330.2999999999997
mampa~event~inqueue~time.HIST-99-percentile: 699.0100000000011
mampa~event~inqueue~time.HIST-999-percentile: 1298.5780000000004

redis~mampa~from~send.HIST-75-percentile: 32.0
redis~mampa~from~send.HIST-95-percentile: 151.0
redis~mampa~from~send.HIST-99-percentile: 133294.30000000008
redis~mampa~from~send.HIST-999-percentile: 136519.942

redis~mampa~from~tell.HIST-75-percentile: 57.0
redis~mampa~from~tell.HIST-95-percentile: 310.3499999999983
redis~mampa~from~tell.HIST-99-percentile: 133891.84
redis~mampa~from~tell.HIST-999-percentile: 136521.681
```

打到200w请求时，问题暴露出来了，这时发现内存飙升！
```
load = 5
mem=15G
```

#### 步骤4：分析内存异常
重复实验几次后，大多数情况下都能重现内存暴涨的情况。于是通过 [MAT分析java内存](http://lyso.me/2014/07/01/bytebuf-memleak) 发现，redis-mampa内部的queue的size暴涨。这个queue是用来缓存pipiline发送到redis-server的命令以解析结果的发回给客户端的，当netty到redis-server这条通道的消息速度比客户端发来的慢时，就会导致queue堆积。

这就需要两个角度解决问题：
1. 需要对queue的size设置上限，在达到上限时降级服务，并打perf-counter
2. 调查为何queue会堆积，因为qps没有达到实验性能


#### 步骤5：queue-size限制
加上 queue size 限制及perf：
```
main.thread=6
redis.thread=4
qps = 10000
```
跑第一次时发现latency很小并且内存未涨，重复实验后又重现压力大的情况，但表现是：绝大多数请求因为redis超时和queue堆积而丢掉了（latency也很大）：
```
redis~mampa~fail~queue~full.COUNTER=761222
```

但查看redis-server的cpu/内存占用情况和机器load情况都无异常，猜测是由于主业务actor和redis-mampa的actor都是基于多优先级版disruptor的，用的SleepingWaitStrategy，这个等待策略先spin100次、再spin并yield100次、然后parkNanos，对CPU占用较大（由于是多优先级队列，不易使用BlockingWaitStrategy），于是将线程数改掉继续试验。

#### 步骤6：调整线程数
```
main.thread=3
redis.thread=3
qps = 10000
```

```
wait~before~process.HIST-75-percentile: 133.0
wait~before~process.HIST-95-percentile: 167.0
wait~before~process.HIST-99-percentile: 175.0
wait~before~process.HIST-999-percentile: 196.942
Request.lifetime.HIST-75-percentile: 131.0
Request.lifetime.HIST-95-percentile: 201.0
Request.lifetime.HIST-99-percentile: 249.71000000000004
Request.lifetime.HIST-999-percentile: 260.913

mampa~event~inqueue~time.HIST-75-percentile: 5.0
mampa~event~inqueue~time.HIST-95-percentile: 29.0
mampa~event~inqueue~time.HIST-99-percentile: 55.0
mampa~event~inqueue~time.HIST-999-percentile: 114.59400000000005

redis~mampa~from~send.HIST-75-percentile: 58.0
redis~mampa~from~send.HIST-95-percentile: 109.0
redis~mampa~from~send.HIST-99-percentile: 156.71000000000004
redis~mampa~from~send.HIST-999-percentile: 171.971

redis~mampa~from~tell.HIST-75-percentile: 67.75
redis~mampa~from~tell.HIST-95-percentile: 116.0
redis~mampa~from~tell.HIST-99-percentile: 167.1300000000001
redis~mampa~from~tell.HIST-999-percentile: 172.0
```

数据表示毫无压力，继续增加qps试试。


#### 步骤7：qps=12000
```
main.thread=3
redis.thread=3
qps=12000
```

```
wait~before~process.HIST-75-percentile: 144.0
wait~before~process.HIST-95-percentile: 166.0
wait~before~process.HIST-99-percentile: 181.71000000000004
wait~before~process.HIST-999-percentile: 195.0
Request.lifetime.HIST-75-percentile: 128.0
Request.lifetime.HIST-95-percentile: 214.0
Request.lifetime.HIST-99-percentile: 296.0
Request.lifetime.HIST-999-percentile: 316.942
```

此时load=0.1，CPU=400%，latency.99在300ms，继续增加qps：


#### 步骤8：qps=15000
```
main.thread=3
redis.thread=3
qps=15000
```
在启动阶段load和CPU较高，latency也比较大，在打到60w-100w的时候latency降下来了：
`mem=1.6G，CPU450%，load=3.0`
`rabbitmq.load=0.29, cpu=450%`

```
wait~before~process.HIST-75-percentile: 186.0
wait~before~process.HIST-95-percentile: 223.54999999999995
wait~before~process.HIST-99-percentile: 235.0
wait~before~process.HIST-999-percentile: 250.0

Request.lifetime.HIST-75-percentile: 140.75
Request.lifetime.HIST-95-percentile: 239.0
Request.lifetime.HIST-99-percentile: 369.84000000000015
Request.lifetime.HIST-999-percentile: 408.797
```

但是latency相对较高，每次请求有两次redis的cmget请求，qps=15000时redis请求为30000，4个线程的实验数据要比这个好的多，所以猜测这里还有猫腻，降低redis线程数继续试验：


#### 步骤9：比较不同redis线程数
```
main.thread=2
redis.thread=2
qps=15000
```

的情况是：

`mem=1.9G，CPU450%，load=4.0`

```
wait~before~process.HIST-75-percentile: 192.0
wait~before~process.HIST-95-percentile: 224.0
wait~before~process.HIST-99-percentile: 241.0
wait~before~process.HIST-999-percentile: 263.942
Request.lifetime.HIST-75-percentile: 201.75
Request.lifetime.HIST-95-percentile: 316.0
Request.lifetime.HIST-99-percentile: 348.0
Request.lifetime.HIST-999-percentile: 383.0
```

而
```
main.thread=2
redis.thread=4
qps=15000
```
的情况是：

`mem=1.5G，CPU450%，load=0.3`
```
wait~before~process.HIST-75-percentile: 183.0
wait~before~process.HIST-95-percentile: 216.54999999999995
wait~before~process.HIST-99-percentile: 378.1300000000001
wait~before~process.HIST-999-percentile: 416.65200000000004
Request.lifetime.HIST-75-percentile: 90.0
Request.lifetime.HIST-95-percentile: 160.54999999999995
Request.lifetime.HIST-99-percentile: 241.42000000000007
Request.lifetime.HIST-999-percentile: 312.942

redis~mampa~from~send.HIST-75-percentile: 53.75
redis~mampa~from~send.HIST-95-percentile: 198.54999999999995
redis~mampa~from~send.HIST-99-percentile: 250.84000000000015
redis~mampa~from~send.HIST-999-percentile: 266.971

redis~mampa~from~tell.HIST-75-percentile: 69.0
redis~mampa~from~tell.HIST-95-percentile: 198.0999999999999
redis~mampa~from~tell.HIST-99-percentile: 262.5500000000002
redis~mampa~from~tell.HIST-999-percentile: 270.0
```

发现增加线程数的影响很小，进一步分析perf-counter，发现了redis的actor压力分布不均的情况，以下是redis线程分别是3、4、5的情况：

```
mampa~tell~actor~RedisMampa~0/3.P0.COUNTER=743969
mampa~tell~actor~RedisMampa~1/3.P0.COUNTER=2833514
mampa~tell~actor~RedisMampa~2/3.P0.COUNTER=743539
```

```
mampa~tell~actor~RedisMampa~0/4.P0.COUNTER=155403
mampa~tell~actor~RedisMampa~1/4.P0.COUNTER=156336
mampa~tell~actor~RedisMampa~2/4.P0.COUNTER=155332
mampa~tell~actor~RedisMampa~3/4.P0.COUNTER=758472
```

```
mampa~tell~actor~RedisMampa~0/5.P0.COUNTER=620549
mampa~tell~actor~RedisMampa~1/5.P0.COUNTER=3686638
mampa~tell~actor~RedisMampa~2/5.P0.COUNTER=620688
mampa~tell~actor~RedisMampa~3/5.P0.COUNTER=620500
mampa~tell~actor~RedisMampa~4/5.P0.COUNTER=620516
```

分析发现数字很有规律，总是有一个actor上落的比较多，其他的比较平均，多的数量是少的数量的(actor个数+1)倍，以上表5个actor为例，可以这样认为，有两种请求，第一种请求平均分给了5个actor，每个62w，第二种请求全落在actor-1上了，共306w，则actor-1上有368w个。然后意识到一种key是带userid的，另一种是appid的，这里测试app只有一个，所以落在了一个actor上。解决方案是，提供可配置的`random-actor`方式路由到不同的actor，继续试验。

#### 步骤10：配置redis-mampa使用random-actor路由消息

```
main.thread=2
redis.thread=5
qps=20000
```
实验结果中redis的actor分布较均匀了。

```
mampa~tell~actor~RedisMampa~0/5.P0.COUNTER=908895
mampa~tell~actor~RedisMampa~1/5.P0.COUNTER=907672
mampa~tell~actor~RedisMampa~2/5.P0.COUNTER=910176
mampa~tell~actor~RedisMampa~3/5.P0.COUNTER=910250
mampa~tell~actor~RedisMampa~4/5.P0.COUNTER=909681
```

此时的latency为：

```
wait~before~process.HIST-75-percentile: 21866.25
wait~before~process.HIST-95-percentile: 25688.65
wait~before~process.HIST-99-percentile: 26831.920000000002
wait~before~process.HIST-999-percentile: 27347.855
Request.lifetime.HIST-75-percentile: 43.0
Request.lifetime.HIST-95-percentile: 106.54999999999995
Request.lifetime.HIST-99-percentile: 190.20000000000027
Request.lifetime.HIST-999-percentile: 239.913

redis~mampa~from~send.HIST-75-percentile: 5.0
redis~mampa~from~send.HIST-95-percentile: 23.0
redis~mampa~from~send.HIST-99-percentile: 80.0
redis~mampa~from~send.HIST-999-percentile: 115.50700000000006

redis~mampa~from~tell.HIST-75-percentile: 7.0
redis~mampa~from~tell.HIST-95-percentile: 27.0
redis~mampa~from~tell.HIST-99-percentile: 75.71000000000004
redis~mampa~from~tell.HIST-999-percentile: 121.94200000000001

mampa~event~inqueue~time.HIST-75-percentile: 13.0
mampa~event~inqueue~time.HIST-95-percentile: 224.64999999999986
mampa~event~inqueue~time.HIST-99-percentile: 291.4200000000001
mampa~event~inqueue~time.HIST-999-percentile: 440.6220000000003
```

可见redis的请求latency已经落回到合理的范围，但`wait~before~process`的时间很长，而且观察rabbitmq的状态发现fetcher模块整体produce的速度只有18000，而fetcher业务模块的disruptor占用情况为：

```
mailbox~occupied~size~Fetcher~0.p1.GAUGE=465
mailbox~occupied~size~Fetcher~0.p2.GAUGE=1859
mailbox~occupied~size~Fetcher~1.p1.GAUGE=533
mailbox~occupied~size~Fetcher~1.p2.GAUGE=1981
```

因此猜测是main.thread成为了瓶颈，于是调整一下线程分配：

#### 步骤11：调整main/redis线程分配

```
main.thread=3
redis.thread=4
qps=20000
```
此时`load=9, cpu=700%`，fetcher模块整体produce的速度基本达到了20000。

```
wait~before~process.HIST-75-percentile: 7854.75
wait~before~process.HIST-95-percentile: 11740.599999999999
wait~before~process.HIST-99-percentile: 16918.420000000013
wait~before~process.HIST-999-percentile: 18196.202
Request.lifetime.HIST-75-percentile: 64.0
Request.lifetime.HIST-95-percentile: 126.64999999999986
Request.lifetime.HIST-99-percentile: 206.84000000000015
Request.lifetime.HIST-999-percentile: 249.79700000000003

mampa~event~inqueue~time.HIST-75-percentile: 17.0
mampa~event~inqueue~time.HIST-95-percentile: 255.54999999999995
mampa~event~inqueue~time.HIST-99-percentile: 374.3900000000003
mampa~event~inqueue~time.HIST-999-percentile: 443.971

redis~mampa~from~send.HIST-75-percentile: 8.0
redis~mampa~from~send.HIST-95-percentile: 37.0
redis~mampa~from~send.HIST-99-percentile: 47.0
redis~mampa~from~send.HIST-999-percentile: 51.0

redis~mampa~from~tell.HIST-75-percentile: 13.0
redis~mampa~from~tell.HIST-95-percentile: 37.0
redis~mampa~from~tell.HIST-99-percentile: 50.0
redis~mampa~from~tell.HIST-999-percentile: 51.971000000000004
```

但`wait~before~process`时间仍然较长，一部分原因是启动时就有堆积的延迟导致的，另外还需继续跟进的方面有：

### 后续工作
1. rabbitmq的CPU占用一直在500%上下，这里是否是瓶颈
2. 能在多优先级队列的disruptor中支持更多的wait策略是否能解决load高的问题

#### 多优先级队列disruptor支持BlockingWaitStrategy

实验验证这种策略发现性能是有一定折扣的，

```
Blocking:  Benchmark/Single Ellapsed: 3007.42ms, QPS: 468862.87. (producerCount=1, ringSize=1024, queries=10000000). 
Blocking:  Benchmark/Multi  Ellapsed: 6067.46ms, QPS: 232398.09. (priorities=[1000, 1000], ringSize=1024x2, queries=10000000).
Sleeping:  Benchmark/Single Ellapsed: 1126.05ms, QPS: 1252227.05. (producerCount=1, ringSize=1024, queries=10000000). 
Sleeping:  Benchmark/Multi  Ellapsed: 1378.33ms, QPS: 1023022.22. (priorities=[1000, 1000], ringSize=1024x2, queries=10000000).
```

但我们肯定达不到20w+的qps，可以先不关心这个，试一下用BlockingWaitStrategy的Fetcher性能：

```
load=5, mem=1.3g, CPU=700%
```

无请求时CPU由60%降到7%

```
Request.lifetime.HIST-75-percentile=57.0
Request.lifetime.HIST-95-percentile=80.0
Request.lifetime.HIST-99-percentile=93.71000000000004
Request.lifetime.HIST-999-percentile=106.91300000000001

mampa~event~inqueue~time.HIST-75-percentile=13.0
mampa~event~inqueue~time.HIST-95-percentile=51.549999999999955
mampa~event~inqueue~time.HIST-99-percentile=94.42000000000007
mampa~event~inqueue~time.HIST-999-percentile=181.56500000000005


redis~mampa~from~send.HIST-75-percentile=5.0
redis~mampa~from~send.HIST-95-percentile=11.0
redis~mampa~from~send.HIST-99-percentile=20.0
redis~mampa~from~send.HIST-999-percentile=24.91300000000001

redis~mampa~from~tell.HIST-75-percentile=7.0
redis~mampa~from~tell.HIST-95-percentile=16.0
redis~mampa~from~tell.HIST-99-percentile=26.710000000000036
redis~mampa~from~tell.HIST-999-percentile=44.76800000000003
```

OK，latency.99降到了100ms。
