title: 追查线上问题之——ByteBuf内存泄露
date: 2014-07-01 21:22:11
tags: 
 - 内存泄露
 - ByteBuf
 - PolledByteBufAllocator
 - Netty
 - NIO
 - jmap
 - jhat
 - MAT
categories: 
---
![leak](http://content.xilu.com/uploadfile/2011/0815/20110815094715719.jpg)

　　最近新上线的异步化服务，查看perf-counter完全没有达到理论的QPS，于是开始在线下模拟线上情况做实验，结果总是符合理论推算，无法重现线上的情况，甚至一度怀疑是hbase服务集群扛不住压力导致的。在无计可施时，在亮老师和欧老师的指点下，开启了这段分析之旅，这里做个记录，方便有遇到类似问题的童鞋们有个参照。

### 看进程的情况：top

`[leo@online:~]# top`

top命令查看这个线程的情况，发现内存占用量很大：

```
Tasks: 254 total,   1 running, 232 sleeping,   0 stopped,  21 zombie
Cpu(s): 22.7%us,  5.3%sy,  0.0%ni, 71.8%id,  0.0%wa,  0.0%hi,  0.3%si,  0.0%st
Mem:  32850724k total, 31968708k used,   882016k free,    88052k buffers
Swap: 12582904k total,   953140k used, 11629764k free, 14371656k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
13205 root      20   0 8120m 3.3g  18m S 120.7 10.7   5890:33 java -XX:...
 5591 root      20   0 8835m 2.3g 9996 S 33.6  7.5   3491:29 java -XX:...
```

### 看gc情况：jstat
`[leo@online:~]# jstat -gcutil 13205 100 10`

```
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25876 24241.376 24954.239
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25876 24241.376 24954.239
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25876 24241.376 24954.239
  0.00  97.03  99.56  99.94  60.00  33453  712.862 25876 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
  0.00 100.00 100.00  99.98  60.00  33453  712.862 25877 24243.470 24956.333
```

　　不看不知道，一看吓一跳：Eden代、老生代全满了，频繁发生Full GC！  

### 看进程内存占用情况：jmap
`[leo@online:~]# jmap -histo:live 13205 | less`


```
 num     #instances         #bytes  class name
----------------------------------------------
   1:         34943     2675903544  [B
   2:       3611235      194315032  [C
   3:       3622541      115921312  java.lang.String
   4:       1037136       29288720  [J
   5:        979922       23518128  java.util.BitSet
   6:        407110       19541280  com.xiaomi.xmpush.thrift.Target
   7:         96961       16289448  com.xiaomi.xmpush.thrift.XmPushServerContainer
   8:         70627        9615944  <methodKlass>
   9:         70627        9534712  <constMethodKlass>
  10:        158114        8854384  com.xiaomi.xmpush.thrift.SubscribeTopicInfo
  11:        239540        7665280  java.util.HashMap$Entry
  12:         76359        6108720  com.xiaomi.xmpush.thrift.XmPushBroadcastTopic
  13:          5016        6029160  <constantPoolKlass>
  14:         89860        5358112  <symbolKlass>
  15:         80553        5155392  com.xiaomi.xmpush.thrift.PushMetaInfo
  16:         69778        5148672  [Ljava.lang.Object;
  17:         72151        4617664  com.xiaomi.xmpush.thrift.PushRegistrationInfo
  18:         92951        4461648  io.netty.util.HashedWheelTimer$HashedWheelTimeout
  19:          5016        4450704  <instanceKlassKlass>
  20:         96884        3875360  com.xiaomi.xmpush.thrift.XmPushActionFetchMessage
```

　　有34943个byte[]示例居然占去了2.5G的内存！得好好分析下这里出了什么问题。<span/>

### 看进程内存详细占用情况：jmap
`[leo@online:~]# jmap -dump:format=b,file=/tmp/fetch.13205.bin 13205`

　　将内存全部dump到本地文件（还是线上机器），然后压缩一下，传输到远程本地机器，并解压： 

```
[leo@online:~]# tar cvzf /tmp/fetch.tar.gz /tmp/fetch.13205.bin
[leo@online:~]# scp /tmp/fetch.tar.gz leo@leohost:/tmp
[leo@leohost:~]# tar -xvzf /tmp/fetch.tar.gz
```

### 分析内存：mat 
　　jhat和mat可以查看jmap的结果，jhat挺弱，用mat可以在eclipse安装插件，也可以在这里[下载mat](http://www.eclipse.org/mat/downloads.php)  

#### Overview 
　　在首页上点击【Open Dump File】，在【Overview】页发现4块大内存占去了绝大部分： 
![image](http://lyso.qiniudn.com/memleak.4.png)  

#### Dominator Tree 
　　在这里可以看得到，前四个是 `  io.netty.buffer.PoolArena$HeapArena  ` 对象。难道我代码里哪里hold住了这么大的内存？印象中没有申请过啊，赶紧去看看吧。 
![image](http://lyso.qiniudn.com/memleak.1.png) 

#### Reference Tree 
　　在任意一个对象上右击，【Path to GC Root】->【with all references】，可以看到下图，哈哈，找到了自己的代码（`com.xiaomi.mampa.redis.utils.Utils`）！ 
![image](http://lyso.qiniudn.com/memleak.3.png)
　　这里有一段这样的代码： 
```
    public static final ByteBufAllocator bufPool = new PooledByteBufAllocator();

    // ...
    public static ByteBuf allocByteBuf(int initialCapacity) {
        return bufPool.buffer(initialCapacity);
    }
```

　　就是这里的问题了，异步pipeline的客户端Redis-Mampa中在序列化前使用ByteBuf对象暂存指令和参数，这个ByteBuf是基于引用计数的，需要用完之后release。后来跟长者陈述这个问题时，长者一拍大腿“哎呀你不早问我！用netty的东西都得自己申请和释放！”。早问你了我不会像这样记这么***深刻***^^。<span/>

### 修复
　　很简单，用完就release呗：

```
    /**
     * Encode and write this command to the supplied buffer using the new
     * <a href="http://redis.io/topics/protocol">Unified Request Protocol</a>.
     *
     * @param buf Buffer to write to.
     */
    public void encode(ByteBuf buf) {
        buf.writeByte('*');
        Utils.writeLongAsString(buf, 1 + (args != null ? args.count() : 0));
        buf.writeBytes(Utils.CrLf);
        buf.writeByte('$');
        Utils.writeLongAsString(buf, type.bytes.length);
        buf.writeBytes(Utils.CrLf);
        buf.writeBytes(type.bytes);
        buf.writeBytes(Utils.CrLf);
        if (args != null) {
            buf.writeBytes(args.buffer());
            args.release();
        }
    }
```

### 验证 
　　好，改完了验证一下。写个循环申请500M内存，分别不release和release，然后sleep住，并用jmap将内存dump出来用mat查看，得到下列两幅图，验证成功。
![不进行release的内存使用](http://lyso.qiniudn.com/memleak.5.png)
![进行release的内存使用](http://lyso.qiniudn.com/memleak.6.png)
