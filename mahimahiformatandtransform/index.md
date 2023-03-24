# MAHIMAHI格式理解和转化逻辑


## MAHIMAHI格式正确解读

> Each line gives a timestamp in milliseconds (from the beginning of the
> trace) and represents an opportunity for one 1500-byte packet to be
> drained from the bottleneck queue and cross the link. If more than one
> MTU-sized packet can be transmitted in a particular millisecond, the
> same timestamp is repeated on multiple lines.

（对啊明明写的是ms怎么后面就一点也不记得了）

mahimahi文件中是一序列的整数，每一个整数代表网络模拟开始以来的时间戳，单位为milisecond毫秒ms，代表着这个时刻会向模拟网络链路中发送大小为1.5KB的数据包，也就是12Kb，这个大小就是MSS（*TCP连接建立时，收发双方协商通信时每一个报文段所能承载的最大数据长度*）
例如一个速度为12Mbps的网络的mahimahi格式的trace即为：0, 1, 2, 3, 4, ...。也就是分别在t=0、1、2、3、4ms的时刻发送了12Kb大小的包，那么反过来算就是12Kb/1ms = 12Mbps的速度了

再例如一个内容为0, 1, 1, 1, 1的mahimahi格式trace file，这就代表着它在t=1ms的时刻发送了四个数据包，那么可计算其速度为4*12Kb/1ms = 48Mbps

## MAHIMAHI trace file转化逻辑和代码，以自定义的trace log file为例

代码直接来源是Pensieve仓库，[traces文件夹下](https://github.com/hongzimao/pensieve/blob/master/traces/norway/convert_mahimahi_format.py)

要点有3：

1. 带宽计算的单位，包括秒s和毫秒ms、比特b和字节B的转化

2. 变量millisec_count会归0，他代表原trace中提供的带宽对应的一段时间，可以理解为以ms的粒度在走完一个receive time。因此每次跳出循环就是判断millisec_count是否超过了这个块的接收时间，如果已经超过了，就不继续写了

3. 变量millisec_time就是一直在递增的时刻，（它不重置0的），当满足条件时，就会写一个或者多个当前的milisec_time。这个条件就是to_send向下取整要大于0。算的上是核心逻辑了，下面展开说

to_send，表示要发送的数据包的个数，满1即发，它是这么计算的：

```python
to_send = (millisec_count * pkt_per_millisec) - pkt_count
to_send = np.floor(to_send)

for _ in range(int(to_send)):
    mf.write(str(millisec_time) + '\n')
    
pkt_count += to_send
```

其中，pkt_per_millisec = throughput / BYTES_PER_PKT，也就是当前这个throughput里，其实对应了多少个12Kb包，每一次随着milisec_cout增加就发一个包，直到发够一个单位1



了然😊

## 关于最后的时间戳并不是原trace的finish_time的时间戳

我开始疑惑了好久，只有一行的话，都能对得上，也就是mahimahi的最后一个时间戳就是原trace 的finishtime（差不太远），但是一旦是多行，两边的时间就对不上了。仔细想想：转换代码里面每次进循环都只关注了tracefile中有吞吐量这一行，在开始时间到结束时间内，它确实可以正确表示，但是我一直忽略了，上一次的finish_time到下一次的start_time之间其实还有个idel_time！！！

理论上讲这个idel_time期间是没有带宽的，或者说不可测，因为都没发送数据。但是mahimahi不会体现这个idel time，他可以说是把所有idel time都给压缩了，变成持续的反映带宽的时间戳。所以两边的时间戳对不上

影响大吗？

---

至此这个数据集转换因该是没问题了
