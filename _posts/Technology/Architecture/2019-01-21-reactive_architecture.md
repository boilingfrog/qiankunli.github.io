---

layout: post
title: 反应式架构
category: 技术
tags: Architecture
keywords: reactive

---

## 简介 （未完成）

从术语上，本文不严格区分反应式编程/响应式编程，从各种材料看，称呼为“反应式”更好。对应的反应式编程的反面——顺序编程/命令式编程 也不做严格区分。

异步分为不同的层次，有代码上的、框架上的、架构上的 [不同层面的异步](http://qiankunli.github.io/2017/05/16/async_program.html)

反应式思想 也分为不同的层次， 有架构层面的，代码层面的——[rxjava1——概念](http://qiankunli.github.io/2018/06/20/rxjava.html)

## 全面异步化：淘宝反应式架构升级探索

[全面异步化：淘宝反应式架构升级探索](https://www.infoq.cn/article/2upHTmd0pOEUNmhY5-Ay)

反应式架构与一般架构相比，其反应体现在：

1. 对用户有反应，对用户有反应我们才说响应，一般我们说的响应，基本上都说得针对跟用户来交互。
2. 对输入有反应，响应系统的输入，也可以叫做消息驱动。
3. 要对失败有反应，应用失败了系统不能无动于衷，等着它挂掉，要有反应。
4. 要对容量和压力变化有所反应，比如说淘宝的秒杀，系统需要反应来保证对用户的响应性，再如那个当流量降下来，将系统缩容，可以节约成本，这也是一种反应。

要做到反应式，需要做到三点：

1. 适应性，也就是发生失败能恢复回来，无论是系统、网络、代码出现了问题都能恢复。
2. 弹性，这点主要是应对流量的变化，弹性的前提是做到可伸缩性 Scalability，从软件设计上，要做到去中心化；同时，在运行时，要感知节点当前的系统负载，将压力往上游进行反馈，做到系统可以感知链路级别的节点压力
3. 消息驱动，有了消息驱动才能比较好的做到上面两个点。在反应式架构里，以前这点叫做事件驱动，后来改为消息驱动，消息驱动强调无阻塞、无 callback，所以不会有线程挂在那里，不会有持续的资源消耗。同时，事件驱动或消息驱动都是异步化，而**异步化会将操作系统中的队列情况显式地提升到了应用层，使得应用层可以显式根据队列的情况来进行压力负载的感知**（PS，常规的同步操作，会让os 线程让出cpu，在os thread struct list 中等着被调度。异步之后，线程都是满负荷在跑）


![](/public/upload/architecture/reactive_architecture.png)

PS：从上图学到的一点是，在反应式架构下， app 并不是直接和组合业务服务交互的，有了网关一层的包装，http 响应本质就是 用tcp连接 给网关写返回值数据而已。我们想象一下一个中间rpc 服务的逻辑

	1. 串联rpc 框架，上游请求序列转换为一个输入参数流，一次调用（比如查询product）视为“进来一个productId”
	2. 对输入数据进行转换，可能涉及到其它rpc
	3. 等拿到结果了，写回响应

其实有点类似 mapreduce， map和reduce 归你写，但什么时候在哪里哪个线程跑map/reduce 你就说了不算了。

反应式架构中的核心概念是“流”，流就是面向数据的顺序串行执行的一系列操作组合，它同传统的编程相比，将业务逻辑导致数据改变，变成了操作改变数据，反过来影响业务逻辑的改变。面向流编程就是面向数据编程。PS：没懂


整个方案对业务架构的升级主要包括编程框架、中间件，以及业务方的升级。中间件的升级，包括服务框架（RPC）、网关、缓存、消息（MQ）、DB（JDBC）、限流组件、分布式跟踪系统、移动端 Rx 框架。这其中值得注意的包括，对服务框架的升级，流式实现将在 Dubbo 3 中放出；DB 中的异步集成使用 Ali JVM 协程或用线程池实现；移动端为了支撑已有的 iOS 应用，淘宝开发了 AliRxObjc 并即将开源。


## 以分支逻辑的替换为突破口来理解反应式编程

传递的命令式编程范式以控制流为核心，通过顺序、分支和循环三种控制结构来完成不同的行为。

顺序和循环 使用rxjava 替换起来比较简单，难就难在 分支/ifelse 的替换。参见[Conditional Logic and RX](https://medium.com/netifi/conditional-logic-and-rx-f6acc0e57a48) 值得读三遍

Conditional logic in RX style programming is a big hurdle for new users. If you’re coming from an imperative background there is a learning curve. Once you get the hang of it though, you can create powerful but simple applications that would be almost to difficult the old way.

对数据组中奇数和偶数进行不同的输出：

	public class EvenOrOdd {
  		public static void main(String... args) {
    		Flux.fromIterable(Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
        	.map(EvenOrOdd::check)
        	.doOnNext(System.out::println)
        	.blockLast();
  	}
  	private static String check(int i) {
    	if (i % 2 == 0) {
      		return i + " is even";
   	 	} else {
      		return i + " is odd";
    	}
  	}

那如果奇数和偶数场景下 调用不同的rpc呢？ 那就是将 数字map 为future。不管符不符合condition，都是一种输出，将分支判断转换为 map映射。那如果不符合条件没输出呢？用filter 操作即可。

Rx-style programming really shines with asynchronous code. 反应式编程 对异步代码很友好。只是在写处理逻辑（回调函数），rxjava 并不保证执行顺序线程是你看到的那样，控制权让出去了。命令式编程控制流排斥异步和多线程

## 剖析响应式编程的本质

[剖析响应式编程的本质](https://www.jianshu.com/p/3bdb8dbaa35c) 要点如下：

1. 响应式编程（Reactive Programming）到底是什么？从名词定义来讲，中文的响应式并没有很好地展现Reactive的本意
2. 传统的**顺序编程**（将顺序编程作为 响应式编程的对立面，有助于理解响应式编程）采用每条指令依次执行的方式，倘若上一条指令没有执行结束，当前的线程就得等着，任你如何提升机器性能还是代码性能，如果本质不变，始终改变不了响应需要等待的现实。若要响应迅速，就得把顺序执行指令的方式换一换——同步换成异步，方法执行换做消息发送，于是乎，我们可以精简地定义：**响应式编程就是异步数据流编程。** 
3. 这其实是一种编程范式，是编程理念的一种思想转型。因为采用响应式编程，我们就不再将软件要处理的业务视为对象，又或者函数，而是直接透析到本质：数据流（Data Stream）。一言以蔽之：万事万物皆为流 [everything is a stream](http://slides.com/robwormald/everything-is-a-stream)。这种流动差不多可以归纳为：`Command -> CommandHandler -> Event -> EventHandler -> Command ...`
4. 响应式编程 和CQRS 不谋而合，按照CQRS的设计思想，任何业务都可以分解为两种形式的消息：Query与Command。
5. 执行Command本身是要改变业务对象值的，然而，如果我们将每次变更都视为是一种“状态的迁移”，然后利用事件去记录每次变更，就可以将可变转换为不变。
6. 响应式编程的设计原则是：

	* 保持数据的不变性
	* 没有共享
	* 阻塞是有害的


 [领域驱动 + CQRS](http://qiankunli.github.io/2017/12/25/ddd.html)

## 待整理

2018.8.24 补充：我们使用rxjava，会发现，它对观察者 定义了接口，对被观察者 也定义了接口， 在两个定义的接口之间（被观察者管发送，观察者管接收，中间可以玩很多花活儿）：

1. 首先是数据流的处理：有了数据流，那么数据的那一套转换、缓冲等可以剥离出来。
2. 同步异步，线程的切换等

**按照程序 = 控制 + 逻辑。逻辑就是被观察者 如何发射， 观察者如何接收，控制则是同步、异步等。再比如数据流转换是逻辑，但n个转换逻辑如何串起来是控制**

2019.1.5 补充 [响应式架构与 RxJava 在有赞零售的实践](https://mp.weixin.qq.com/s?__biz=MzAxOTY5MDMxNA==&mid=2455759277&idx=1&sn=3096d192749deadeab3136751579493a&chksm=8c686f88bb1fe69e8182cf4659915a415b7d340c4257ab5d6a794283659906142837f824dd18&mpshare=1&scene=23&srcid=0105rUx9Zxj9jHjE1TfysyMX%23rd) 响应式架构（vs 微服务），响应式编程 。可惜还是找不到感觉。

微服务之间的通信的最佳机制就是消息传输。如上文所说，服务之间的异步边界能够在时间和空间两方面进行解耦，能够提升整体系统的性能。