---

layout: post
title: 以新的角度看数据结构
category: 技术
tags: Algorithm
keywords: 数据结构

---


## 前言（未完待续） 

每本《数据结构》书上的主线都跑不了以下几点，时常回味一下，还是蛮有好处的。

数据结构分为逻辑结构、存储结构以及对应结构的数据运算，比如，你可以用一个数组表示一个图，也可以用链表存储一个图（或者一个图中包含数组和链表）。

存储结构主要包括：

1. 顺序存储：把逻辑上相邻的元素存储在物理位置上也相邻的存储单元里，元素之间的关系由存储单元的邻接关系来体现。其优点是可以实现随机存取，每个元素占用最少的存储空间；缺点是只能使用相邻的一整块存储单元，因此可能产生较多的外部碎片。

2. 链接存储：不要求逻辑上相邻的元素在物理位置上也相邻，借助指示元素存储地址的指针表示元素之间的逻辑关系。其优点是不会出现碎片现象，充分利用所有存储单元；缺点是每个元素因存储指针而占用额外的存储空间，并且只能实现顺序存取。

3. 索引存储：在存储元素信息的同时，还建立附加的索引表。索引表中的每一项称为**索引项**，索引项的一般形式是：（关键字，地址）。其优点是检索速度快；缺点是增加了附加的索引表，会占用较多的存储空间。另外，在增加和删除数据时要修改索引表，因而会花费较多的时间。

4. 散列存储：根据元素的关键字直接计算出该元素的存储地址，又称为Hash存储。其优点是检索、增加和删除结点的操作都很快；缺点是如果散列函数不好可能出现元素存储单元的冲突，而解决冲突会增加时间和空间开销。

逻辑结构主要是：

1. 集合结构中的数据元素之间除了 “同属于一个集合”的关系外，别无其他关系。
2. 线性结构结构中的数据元素之间只存在一对一的关系。
3. 树形结构结构中的数据元素之间存在一对多的关系。
4. 图状结构或网状结构结构中的数据元素之间存在多对多的关系。

每种逻辑结构包含一些基本的运算，包括遍历，增减节点等。

## 队列

教科书上一说队列，就是四个字“先进先出”,这四个字是无法表述队列的巨大作用的.

### 合并请求的一种实现

假设一个方法或类负责管理一个资源,在多线程环境下,这个类便需要"线程安全"

1. 将这个类改造成线程安全的类
2. 调整这个类的调用方式.利用生产者消费者模式,所有想要使用这个资源的的线程(生产者)先提交请求到一个队列,由一个专门的线程(消费者)负责接收并处理请求.

### 轮询的一种实现

假设我们有一个集合，对集合中的元素实现轮询效果。

1. 我们可以标记一个index，记住上一次使用的元素，进而实现轮询。
2. 用环形队列存储集合，即天然具备轮询效果。

这两种方式，在多线程环境下，还需注意线程安全问题。

## 图

深度优先和广度优先遍历，{初始状态，目标状态，规则集合}在寻找最佳策略上的应用

## 数据结构的"基本类型化"

较早出现的编程语言,比如c语言,基本类型只包括:int,char,long等,string,map等则需要引用库.而对于新兴语言,比如go和python等,则在语言层面支持string,map等复杂结构.这一趋势,甚至扩展到了一些内存数据库.


