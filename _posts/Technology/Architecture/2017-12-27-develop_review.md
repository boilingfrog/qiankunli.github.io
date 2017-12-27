---

layout: post
title: 2017软件开发小结—— 从做功能到做系统
category: 技术
tags: Architecture
keywords: coder

---

## 简介


做功能，本文指基于一个现有系统进行增强，有产品经理描述需求。此时，无论上功能描述上，还是代码实现上，都有清晰的边界。特定到web后台开发，Controller/Action==> service ==> dao 这种本质上面向过程的代码，大部分细节都已确定。

做系统，**此处特别指没有产品经理进行产品设计的系统，比如系统本身来源于技术需求，使用方也主要是程序猿。**

## 做系统

做系统主要有以下特点：

1. 做什么，大概知道，但也不是特别知道。未来如何演变，不好估计。
2. 碰到问题有哪些，其中的关键点有哪些也不是特别清楚
3. 有些系统需要一定的理论支持，比如ABTest系统
4. 假设需要一个操作界面，很实际的问题是：界面应该长什么样子。
5. 与其它系统的关系

这些问题会带来一些实际的困难

1. 一个问题通常有多种解决方案，不知道“痛点”，就无从取舍。譬如，假设每年GDP要保xx，那么环保是放松还是加紧些，就有了决策的依据。
2. 界面 ==> Action ==> Service ==> Dao ==> 数据库。若没有经过充分抽象，界面极其容易成为数据库表的增删改查。
3. 对系统的关键点不熟悉，导致：

	* 很容易陷入求大求全的误区中
	* 强行要求支持一些不必要的特性
	* 对一些细节纠缠不清


经过一年的实践，笔者得到以下经验：

1. 不要吝惜在设计方案上的时间，尽可能召集所有的相关人员开会，汇总不同角度的看法和反馈

	* 尽可能的从网络上搜寻现有系统的相关材料，了解系统有什么组件、组件之间的关系。技术分享非常多，大部分系统都有blog，汇总他们，有助于理解系统，有助于说服自己、说服别人。
	* 耐心，不要试图从一次会议中得出结论。笔者为了制定一个协议，一共开了四次，分别是：这个协议要考虑哪些问题，哪些问题必须实现，哪些可以牺牲一下；现有几个方案的分析；自定义初步方案评估；非协议问题以及其它细节。
	* 这同时也带来一个问题，说服大家达到一致。而说服的过程，本身就要求我们将各种问题想的清楚一点
	* 如果你完全不清楚一个系统应该是什么样子，那么软件工程中的UML图是个有力的工具，让别人快速看懂你的想法，在沟通的时候非常重要。
	* **问题从“从何解决问题” 变成“这个系统可能有什么问题”，是的，提出问题非常重要。**
	* 很多事不要一个人担着，思维有局限，心情易沮丧，要跟人聊天。
	
2. 与耐心对立的是，尽快实现一个小版本，去实现当初的一些想法

	* 因为最初需求不明确，修正不可避免，**但如果当初设想的基本理念被推翻，则意味着整个过程是失败的。**比如文件上传中对netty + tcp + 流式上传方案的选用；补丁系统中下发条件关联到组件上而不是文件上。
	* 代码的编写，要求你精确一些细节

3. 放弃尽快完成任务的想法（做功能的时候通常是这样），分析 ==> 开发 ==> 再分析  ==> 开发 才是常态。**反过来说，失败的设计也是有有意义的，因为当时的设计也是有理由支撑的**。笔者曾耗费五六千的代价报班学习吉他，所有成果是两首生硬的弹唱。毫无意义么？也不是，笔者以后看见吉他再也不后悔没学了。
4. 界面设计，通常不能做成数据库表的增删改查，它们之间需要抽象。作为一个后台开发，写html代码或许没有吸引力，但是给用户的操作界面，最终反映了你对系统的理解，为抽象提供思路。操作界面的优劣最直观的方法就是：易懂，操作能少一步就是一步。
5. 不要因自身的技术背景影响到技术选型。笔者作为后端开发，前端代码水平有限，而在实现系统时，一些前端效果的实现又需要等客户端排期，为此后来干脆自己操刀前端代码。结果就是，界面总是感觉土。界面土没什么，若是因为自己技术背景影响到方案选型等，后果就比较严重。

你也许看到了，这些经验彼此矛盾。

## 其它

1. 推广最难，除了直接给用户使用的业务系统，为提高效率开发的支持系统，往往在推广时碰到很大阻力。

	* 系统不够易用，带来的学习成本大于节省的时间。为此，做系统是一方面，如何一键式的使开发转到新的系统上来也非常重要。因此maven plugin、archetype 等工具也要了解下
	* 人的惯性比想象的来的大

2. 维护时间最长。

	* 详细的用户文档、常见bug文档等要准备充分
	* 系统在设计时，预留调试bug的日志、查询接口等

3. 去看博客，听技术分享。你或许不知道细节，但不能不知道一件事有这样一种方案。
4. **每一个种选择都有利弊可以分析，分析它，然后告诉大家，共同讨论。有时候，很奇怪的是，一个重大决策（通常都意识不到），往往不经分析，直接就开始干了。**笔者在下象棋的时候，发现若是在走每一步之前，都先分析下对方为什么走这一步，则胜率会提高很多。但仅仅这一个简单的思维习惯，都难以坚持。

## 小结

1. 本文的大部分内容，课本上都讲过。但就像我们看别人的代码一样，表面上平淡无奇，但只有亲身实践一次才知道，实际上充满了利弊权衡，真的有可能会做错。
2. 最重要的永远是人，有句话叫“你知道了这么多道理，还是过不好这一生”，人在实践一个系统的过程中，正确的认识到所处的阶段，进而调整心态与认识。**这种驾驭自己的心理体验，比实际的技术知识要宝贵的多。**
