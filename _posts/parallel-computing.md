---
title: ParallelComputing
date: 2023-01-08 18:34:22
tags: [misc, knowledge]
categories: [computer-science]
---


# Why Use Parallel Computing?
blabla.
但真正让我感受到计算能力的有限，是在一次物理大作业中。具体的内容记不清了，但反正需要matlab写出三重for循环，来找出极值。然后CPU就开始疯狂转了，等了老半天才出结果。
虽然这个作业能不能并行化，我也记不清了，但是，它让我体会了电脑算力不如我想象的高（我还用的是笔记本电脑，性能比较差）。以前我真的认为要很大的数据规模，才会需要服务器，超算。

直到后来更加系统学习了算法的一些知识，每次估算算法时长时，才发现，CPU的几GHz其实也挺小的。算出某个算法要几个小时才能跑完时，就会想，CPU如果再快个几倍该多好。

semaphores https://greenteapress.com/wp/semaphores/
java https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html book ... in practice

books, need to read?
https://www.amazon.co.uk/Introduction-Parallel-Computing-Ananth-Grama/dp/0201648652/ref=sr_1_2?ie=UTF8&s=books&qid=1279904224&sr=8-2
https://www.amazon.co.uk/Parallel-Scientific-Computing-MPI-Implementation/dp/0521520800/ref=sr_1_3?ie=UTF8&s=books&qid=1279904267&sr=1-3

# 一些演变过程

参考《Java Concurrency in Practice》第1章。

最初，只存在单进程。后来，操作系统引入多进程，多进程可以简化代码编写的难度，因为你可以只对一种任务写一个程序。各个程序间可以交互，而调度交由操作系统。
-> 比在单进程中塞各种任务更容易。（此时串行的基本单位是进程，所以这里说“单进程”）

后来出现多核，多处理器，单个程序也可以不串行，也就是多线程。不仅是提高资源利用率，而且把一个程序拆成多个子任务，每个任务的逻辑单独编写，必要时再交互，也可以简化程序逻辑。
-> 比在单线程中塞不同子任务更容易。（因为此时有多线程了，串行的基本单位是线程，所以这里说“单线程”。实际要表达的是一样的。）

但由于操作系统也是慢慢进化的，早期的操作系统通常会将进程中可创建的线程数量限制在一个**较低的阈值**内，大约在数百个（甚至更少）左右。因此，操作系统提供了一些高效的方法来实现多路I/O，例如 Unix 的 select 和poll 等系统调用，要调用这些方法，Java 类库需要获得一组实现非阻塞 IO 的包 (java.nio）。然而，在**现代操作系统**中，线程数量已得到极大的提升，这使得在某些平台 上，即使有更多的客户端，为每个客户端分配一个线程也是可行的。

也就是说，理论上肯定是多线程+阻塞I/O更好（指线程中使用BIO），可以降低开发难度，线程调度不需要我们操心。但现实是，过于多的线程就可能带来问题。线程使用的内存，线程上下文切换的CPU开销，在“过于多的线程”这个场景下，可能就是内存占的很多，处理速度也不快。可能使用NIO效果会更好，当然，是在某些场景下，无法绝对论。所以，NIO还不会被淘汰（虽然它真的有点复杂，不好理解，又是考点😂）。


# ThreadLocal

Java中线程封闭里有个特别点是ThreadLocal，它就是简化一下写法，并不是什么黑魔法。

首先，理解下ThreadLocal的原理，你使用ThreadLocal::get/set时，内部是去取该线程的ThreadLocalMap，注意Map是存在Thread里的，set就是set值到这个Map，kv是<this（指ThreadLocal对象）, value>，那么也说，一个ThreadLocal对象是同一个没错，但实际get出来的是从某个Thread对象的ThreadLocalMap里拿出来的value。（而不是一个ThreadLocal里存thread到value的映射）

需要注意，每个Thread里对ThreadLocal对象做set，是线程隔离了，各个线程之间不共享。但要是你set同一个value，那本质上还是多线程共享同一个对象。

那么，什么时候用的上ThreadLocal呢？为什么要用它，而不是每个线程都维护一个自己的对象？

网上大部分的ThreadLocal举例都是什么UniqueId、DateFormat，它们完全体现不出ThreadLocal的优点，反而像画蛇添足，原子变量都可以的为什么非要ThreadLocal写UniqueId，每个线程就可以有自己的成员变量直接new DateFormat不就行了，为什么要ThreadLocal来创建DateFormat。

我们先抽象一下场景，假设Thread内会创建某个对象，不管合不合适，我们将它存在Thread类的成员变量里，那么，多线程是new 多个Runnable或者多Thread，才能做到线程之间互不干扰。

如果是create一个Runnable，多线程都由这一个Runnable初始化，那这里就有限制了，多Thread实际用的一个Runnable对象，那多线程肯定就互相干扰了，这时候把Runnable里的成员变量直接改为ThreadLocal的，就可以立马获得线程安全的程序。这是一种ThreadLocal的好处，它可以简单地将单线程程序改成多线程程序。

再考虑这样一个场景，我们有多个函数，它们不是Thread/Runnable的函数，是别的类的，它们的输入都是context，每个线程单独一个context，那使用方式就是：
```
context = new Context
A.func1(context)
B.func2(context)
...
```
那如果你不想这么频繁传递，比如，`A.func1()`这些函数内部可以写为：
```
context = ContextTL.get()
```
它们就自然获得该线程的context，那么这个ContextTL自然用ThreadLocal来做，就可以达到目的。使用方式就变成了：
```
context = new Context
ContextTL.set(context)
A.func1()
B.func2()
...
```
单实例变量或全局变量共享，就是这么个意思。没有ThreadLocal的话，set 类static变量当然是线程不安全的，有了ThreadLocal就可以。在线程的角度看来，就像是单线程的情况下，去set一个跟别的对象共享的变量。
