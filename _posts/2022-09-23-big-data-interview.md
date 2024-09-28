---
title: BigData Interview Questions
date: 2022-09-23 14:27:15
tags:
---

涉及大数据的面试题，通常是没法写代码的。当然，也有部分题目本质上是归并、堆或是快排等算法，那就还能写写。

本文讨论一些常见试题。

## 分治

常见的就是给定一个大数据集，但是内存小，不能做到一次性load数据并处理。例如，给1G的文件，内存限制在1M，等等类似的条件。

大数据必然是“分治”，大数据都要能拆成小数据。所以，遇到大数据题目，可以先假定能拆分出小数据，即使还不知道应该hash拆分还是别的算法做拆分。（当然，通常都是hash算法拆分。）

那么，问题就被分割了。我认为应该更早去考虑小数据集的部分，跳过拆分。因为，在考虑小数据集如何处理时，你会想到小数据最好是什么样的组合形式。你就可以按照这个形式去拆分，先拆分可能可能会发现拆的并不好。

比如，word count类题目，你自然希望小数据里可以直接包含某个word，这个word不在别的小数据集中出现。那么自然的，拆分就应该用hash算法，它能保证一个word都被hash到一个片里。

再比如，如果你是希望小数据集之间能有一定顺序，可能按首字母来排序，那么切分数据集时就应该按着首字母来分出26份，这里就不该hash算法，而是类似group by了。（当然，hash是更常见的，无论是题目还是生产中。）

大部分题目，在拆分为小数据集后，就可以默认小数据集满足内存要求，不会OOM了。也就是只做一次hash拆分，更进阶的还要考虑“一次切分后，有些小数据集仍然过大”的情况。

很多网上的答案都是再hash，我认为这里想的有点少了。往MapReduce需要处理的特殊情况那边想，可能更好。比如，Map可能可以加Combiner，Reduce的分配可以调整，多级Reduce，Reduce操作改进到迭代式先处理一部分就扔掉，再或者使用流式等架构，等等。

## 分布式锁

分布式锁不是说锁是分布式的结构。是说如果系统是分布式的，进程间（无论是同一主机上还是多主机）要保证某些资源的独占，就得有个外部工具帮忙来指挥。硬要系统内部做，不引入外部工具，也不是不行，但是从工程角度来看，肯定是直接利用现成工具更好，而且也减轻代码复杂度。（虽然大家经常先用zookeeper，随后又嫌弃它，又移除zookeeper。但前期还是需要zookeeper，毕竟搭建快。）

常见分布式锁就是zookeeper和redis。

具体看算法总结 [abs algo]({{ site.baseurl }}{% link _posts/2022-10-07-abs-algo.md %})。这里后续贴一点常见的具体面试题吧。

## CAP定理 theorem/principle

我的记忆中一直是都满足P，只是看CP还是AP。有一天朋友跟我说有CA，我就混乱了。所以来重新学一遍CAP吧。

注意大前提：分布式系统， a distributed system with replication。

- Consistency (C): Either get the most recently written data or get an error for each read
- Availability (A): Every request can get a (non-error) response, but there is no guarantee that the latest written data will be returned.
- Partition fault tolerance (P): Although any number of messages are lost (or delayed) by the network between nodes, the system continues to operate

consistency一致性很好理解，分布式多副本，就需要考虑读写同一个分片的不同副本会不会有问题。强一致性协议的系统，当然是保证C的，例如paxos，raft。弱一致性协议当然不保证C，例如gossip。需要注意，强一致也不是说每次read都能读到有效数据，如果副本异常，你报错，那也是合理的。起码不会读出更早的数据，用户会当作自己读到的是最新的数据。

availability可用性，很多强一致性系统也说自己高可用，但其实也没那么高可用。这个A说的可用，并没有特指什么东西。系统可以更随便，比如，就是只要能返回给你数据就行，旧的我也给；又或者，系统给你回复，但是很慢很慢才回，但我也没拒绝你的request，也没报错，你也不能说我不A。

partition tolerance分区容忍最难懂，因为如果分区，有中心的系统找不到中心的一个分区必然就挂了，这么说好像就都没有P了。
P是指网络不可信了，信息会丢，paxos/raft等协议也提到过这个概念。

很多地方将CAP三个东西分开讲，也有人说某某系统是CA的，说的好像三个任选其二。但下面这个[理论](https://github.com/henryr/cap-faq#10-why-do-some-people-get-annoyed-when-i-characterise-my-system-as-ca)我觉得更有道理一点：
```
Possibility of Partitions => Not (C and A)
```

只要你会面临分区的问题，你就无法同时保证C和A。而不是去讲那个系统能保证P。如果你非要讨论单机系统，那当然没有分区问题，它CA都满足，那当然可以。（但我们的讨论前提是分布式）

