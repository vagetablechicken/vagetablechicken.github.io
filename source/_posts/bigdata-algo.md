---
title: BigData Algo
date: 2023-09-02 16:40:22
tags: Algo
categories: BigData
---

一些通用的大数据、分布式等方面的算法总结。单独出一篇文章，因为这类算法都是通用的，但具体到各个系统，实现方式可能不同，本篇只总结算法思想，可能伪代码实现最基础的算法，必要时会给出现实系统的参考链接。

## 一致性哈希

哈希可以说是出现在大数据的方方面面了。一到切分，基本都是它。一致性哈希，先说哈希，也就是虚拟哈希空间，有2^32个片，叫哈希环。

然后哪个服务器负责处理哪些哈希片，是靠映射。具体方法没有规定，比如，可以用服务器ip来hash，还是2^32取模，得到的数字，它就处理哈希环上对应的片。数据来了先hash取模，得到的值然后顺时针找可以服务的服务器，就发给它。

![chash](./bigdata-algo/consistent-hashing.jpg)

如果中间某个服务器down了，反正算法是往后找到第一个服务器，它还是会被处理。当然这个就要求“数据可以被任何server处理”，不能是特别的。比如，相关数据只在某个节点，其他节点获取不到，处理条件都不满足。当然这是理想条件，现实很多也不满足，但它们涉及到的数据也可以是多节点都存有备份，但又不至于所有节点都需要。这个也等于缩容，受影响的数据是“会落到这一下线节点的数据”，我们称为nodeB，将前一个节点nodeA的末尾片记为end1，当前节点末尾片为end2，也就是`(end1, end2]`区间的数据原本应该找这个节点B，但它下线了，就会变成去找下一个节点nodeC。其他片上的数据不会受到影响。虚拟哈希环不会改变大小，永远的2^32片。如下图左边所示。

而扩容就在hash环上加一个node，原本`[start, end]`会找nodeD，你在nodeD前面塞了个nodeE，那么`[start, nodeE_end]`这部分数据就是受影响的数据，原来要找nodeD，现在变成nodeE。如下图右边所示。

![scale](./bigdata-algo/upscale-downscale.jpg)

优点不难猜测，节点上下线只会影响局部数据，这已经是很大的优点了。不能傻到还在N个节点，就hash(req)%N吧，增加节点你还需要把N改成N+1。而且它会改变很多请求的路由，缓存都不无法利用。（当然，要是系统就是这样的简单，啥也不要求，那也不用考虑改变了，简单就是最好的。）

https://interviewnoodle.com/how-to-use-consistent-hashing-in-a-system-design-interview-b738be3a1ae3

但这还有优化空间，因为在前面所说的算法中，nodeX总是要分到连续的多片，现实中是很难保证每片都差不多热度，也就更不能要求每个node的请求都是均匀的。而且节点数量改变时，在现实条件下，上下节点是要改变很多东西的，可能你要将一些数据复制到节点中才能正常服务，也叫做Node rebuilding。它是一个很重的操作。

为了解决，又引入“虚拟节点机制”，就是hash环上不是物理server node，而是每个物理server都膨胀出，node0-0、node0-1等等的虚拟节点，这样虚拟节点打散分布在hash环上，倾斜可能性就小些。不再是一个node去处理连续的一大片区域。如下图所示。

![vnodes](https://miro.medium.com/v2/resize:fit:720/format:webp/1*OAiDSxqJBpVkT55fK7n0oA.jpeg)

向图中展示的一样，一个server node处理离散的几个片，那么每个server的负载均衡就很好操作了，比如，我可以让1个server服务3个高热度的，其他server负载10个低热度的。不用费心去在环上取一个很好的位置，才能把连续的几片服务好，有变化时更是运维难度大。而迁移片也是只对这么一个小区域，粒度更细，rebuild压力也就越小。

也有人说Vnodes负载均衡的情况还可能是让性能好的机器服务更多、更热的片，小机器就混一混。机器规格不一致的时候，确实是个好处。不过我实际在工作中挺难看到不一样的机型在同一个集群。可能现在机器越来越便宜了吧，以前还是很看重成本的，甚至看到机器闲置会进行降级。

现在来讲讲如何代码实现一致性哈希。

### 简版

简单版本，我们只做哈希环，然后node负责连续分片。
```
def hash(key):
    return hash(key) % 2^32

map<hash_value, node> ring
# map key可以不是范围，而是end分片，这样可以用lower bound来找到第一个大于等于req_key的node
# 但原生Python不支持，得新建class
# Java ceilingEntry和C++ std都可以直接用

def get_node(key):
    hash_value = hash(key)
    return ring[hash_value]

# main
req_key = hash(req)
node = get_node(req_key)
node.handle(req)
```
```{note}
如果存在一群相同元素，那么 lower_bound 和 upper_bound 就可以找到这群元素的上下界限，前者指向下界限，用 lower 表示，后者指向上界限的后一个位置，用 upper 来表示。也就是说，lb是大于等于key的第一个元素，ub是大于key的第一个元素。
```

### 开源实现

#### Guava consistentHash
Guava库中的[com.google.common.hash.Hashing.consistentHash](https://github.com/google/guava/blob/f491b8922f9dc8003ffdf0cbde110b76bcec4b6e/guava/src/com/google/common/hash/Hashing.java#L660)就实现一致性哈希算法。一般使用方法是，你有一个key，用Hashing中提供的某种hash算法来hash这个key（比自行提供hashCode会好，提供了murmur之类的算法），得到的HashCode去调用consistentHash方法，再加上一个bucket数据限定，就可以得到一个bucket的index，也就是说，你的key会被分到这个bucket里。

但仔细看它的consistentHash函数，它压根儿没做一致性哈希理论的那些事，完全没有哈希虚拟空间的事儿。

理论是源于此文章https://dl.acm.org/doi/10.1145/258533.258660，一致性哈希的根本定义是“对所有的 Buckets 和 Items 应用相同的哈希函数 H，按照哈希值的顺序把它们排列到一条线上，然后将距离 Bucket 最近的 Items 都放入该 Bucket 中”，哈希环是一种实现方式，但不是唯一的实现方式。

但Guava一致性哈希算法实现者的文章，https://arxiv.org/ftp/arxiv/papers/1406/1406.2294.pdf，批斗了一波原版，提出了新的“jump consistent hash”，不用存server/bucket服务哪些哈希片，速度还快。但它是有条件限制的，buckets必须是连续，否则这个index刚好是下线了的node id，那肯定不行。如果要用这一套，必然是得把bucket抽象化，也就是Vnodes那一套。

先只看JumpConsistentHash。首先，先抛弃一致性哈希的经典哈希空间，抛弃空间与真实bucket的对应关系，我们只去想这个算法要干什么，就是想将input放进bucket里，想均匀地放，当然也不能乱放。不乱放是指，key=k1，那么k1在bucket数目恒定时，就应该永远被放入同一个bucket，不然就是随意乱放了，random放，更甚者循环放就行了。

规律总结就是，记`ch(key, bucket_size)`为一致性哈希函数，那么得有：
- ch(k, 1)=0，即所有key都放bucket0，毕竟总数只有1个bucket
- In order for the consistent hash function to balanced, ch(k, 2) will have to stay at 0 for half the keys, k, while it will have to jump to 1 for the other half. In general, ch(k, n+1) has to stay the same as ch(k, n) for n/(n+1) of the keys, and jump to n for the other 1/(n+1) of the keys. ch(k,2)的情况，则是分一半流量给新增的bucket1。ch(k, n+1)对比ch(k, n)则是，n/(n+1)的key完全不变化，1/(n+1)的key，它们的bucket变更成n。（必须n/(n+1)部分是完全不变化，bucket值都不可改变，不然就算不得一致性哈希了，就是随便乱来算法。）而既然是最后还是要key平均进入bucket，当然是得每个bucket里抽出一部分来，放进新bucket。

这样，就可以用ch(k,n)来描述ch(k,n+1)了。也就是说，ch(k,n+1)计算时，都是先用ch(k,n)计算出来，但有一部分key应该考虑分给新bucketn。从一个key的角度来看，它就是在不断jump，它在ch(k,1)时，就是0，然后在ch(k,2)时，就是0或1，满足条件，它就应该到1那儿去，然后在ch(k,3)时，它又该考虑是在bucket1还是跳到bucket2，以此类推。还真是名副其实的jump。

![jump](./bigdata-algo/jump-chash.jpg)

图中，从左下框架上来看，只有?函数安排合理，就能得到一个非常简单的一致性哈希算法实现。那么，关键又变成了什么样的函数才能做到。假设k2输入，返回值是恒定的0.1，那么2个桶，它就会被判定去桶1，3个桶就去桶2。但这么就搞得只有小key才会挪动，不够随机。所以这里的函数一般是随机函数，但由于我们必须保证原则“bucket数量不变时，k应该对应唯一一个bucket index”，所以这个随机不是真的随机。所以实现上，会拿key本身作为seed，它对应不变的一组数字序列，是可预期的。所以对一个key来说，在bucket总数不变时，结果是一样的。

文中还做了进一步优化，大概是能跳过一些没必要的j loop，毕竟j越大，能跳的值就越少了，有些都不太需要处理这一遭。没有仔细看，有空再啃一下原文，可以参考 https://zhuanlan.zhihu.com/p/625123381看一看。

#### Ketama

前面那个属于邪教了，课外读物，当然很厉害。真的去实现理论算法的，较为有名的一种实现叫Ketama算法，该算法最初是由Last.fm的程序员实现的并得到了广泛的应用

标准ketama好像是https://github.com/RJ/ketama，只有c实现，其他语言都是调用c库。本质没什么特别的好像，可能就是较早的广泛的算法实现。

https://github.com/ultrabug/uhashring 是纯Python实现，其中也包含ketama。特点是，中途可以加减node，还有weight，weight就是大的vnode多一点，这样key会较多的到weight高的node上。

redis哪里用到了一致性哈希？redis cluster？
