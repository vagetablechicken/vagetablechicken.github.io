---
title: mvcc
tags:
---

# MVCC

> [Multiversion concurrency control - Wikipedia](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)

> Multiversion concurrency control (MCC or MVCC), is a non-locking concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory.

简单来讲，MVCC就是想去掉锁，否则为了事务和正确性，必须要锁，性能会低。MVCC在各个数据库的实现不一样，细节上有差异，但是大体思路是一样的。
每个database object，我们简单理解为一行，并发读写这一行就得有读写锁。如果想要不加锁，就需要有一些额外的信息来保证正确性。Multiversion就是指一行有多个version，在一些时候我们就可以直接读旧行，而不用加锁。
> When an MVCC database needs to update a piece of data, it will not overwrite the original data item with new data, but instead creates a newer version of the data item. Thus there are multiple versions stored. The version that each transaction sees depends on the isolation level implemented. The most common isolation level implemented with MVCC is snapshot isolation. With snapshot isolation, a transaction observes a state of the data as of when the transaction started.

也就是说mvcc是snapshot isolation。

一种简单的实现是，每个version都有a Read Timestamp (RTS) and a Write Timestamp (WTS) 。维基百科的例子相当的抽象，也完全没有WTS的用法。

找更简单的解释

https://vladmihalcea.com/how-does-mvcc-multi-version-concurrency-control-work/

每一行多了两个列，Xmin和Xmax，值是记录transaction id，min是写它的trans id，max是删除它的trans id。也就是一个生命期的起点和终点，这样就可以判断这一行是否可见了。在min和max之间的就能看到这一行，否则就是看不到。

![](https://vladmihalcea.com/wp-content/uploads/2017/03/mvcc_insert.png)
假设我们使用隔离等级是read committed，当我写入一行，我就以我的trans id 1写入Xmin，我事务内的读写大概是undo log to capture uncommitted changes，刚好别人看不到，但我这个事务内部是有变化的。
假如此时有个id更大的事务trans id 2，它尝试读出这一行（比如select where），没提交的当然读不到。
如果我已提交，无论我在哪个id的事务（大于或小于Xmin）里都应该读到，因为这是read committed等级。

也就是MVCC就是在uncommitted期帮助我们不用锁就能安全读写。committed的，根本不看Xmin和Xmax，直接读就是了。

如果隔离等级是REPEATABLE READ or SERIALIZABLE，这个就不太一样了，毕竟上面那个case就应该变为trans id 2永远读不到trans id 1的改变，因为要保证trans id 2的可重复读。那我们记时间也是可以判定的？只要id的start ts早于行数据中的Xmin，就可以读到，否则就是读不到。这样就可以保证可重复读了。

![](https://vladmihalcea.com/wp-content/uploads/2017/03/mvcc_delete.png)
删除

# database isolation levels

多事务并发，就需要db来选择“事务与事务，能不能看到对方的修改”。这个选择就是isolation level。isolation level越高，越能保证事务的正确性，但是性能也越低。几个隔离级别总是记不住，还是用例子来记忆。中文名字更是抽象，也不用强行记忆了，只要能说出现象，我认为就可以了。

如果事务都是在读，当然大家无事发生，核心就在写上。

最不操心的做法当然是，写就及时生效，什么额外东西都没有。那这样的现象就是，T1写，T2就能看到T1的修改。这个隔离级别叫做Read Uncommitted。（就是read到了uncommitted的东西，简单直白）这个隔离级别最简单，但是也最不安全。不安全在于，T2可能看到T1的修改，但是T1还没提交，T1可能回滚，T2就看到了不该看到的东西，也叫dirty read，没收拾干净的感觉。

那db稍微照顾一下，事务只能读到已提交的，避免它读到未提交的东西。db付出的代价就是要存下两个版本，一个是已提交的，一个是未提交的，应该让别的事物去只读已提交的版本，未提交是当前事务用的。这个隔离级别叫做Read Committed。它当然解决了前面的dirty read，但是也有问题，假设T1读了一次，T2就写提交了，T1再读一次，就会有两个不同的值。这个现象叫做不可重复读（non-repeatable read）。

如果你无法接受不可重复读，那db就要再辛苦一点，保证单个事务T1内读数据不会变。但这里不是什么整张表都snapshot下来给T1，前面说的都只针对一行，如果别的事务T2加了几行或改了别的行，逻辑上当然不应该被限制，那么，T1做范围查询在T2前和T2后就有不一致了。比如T1查top1，T2加了新top后，T1再查一次top1，结果就变了。这个缺点不再是行级别的，而是范围级别的，叫做phantom read，名字也没什么根据，不用纠结。反正它就是在说repeatable read在范围查询上还是有问题。

那db保证范围查询不会变，这个隔离级别就叫做Serializable。它的代价就是，如果T1在T2前后都做了范围查询，那么T2就不能在T1前后做范围查询了。这个串行化的名字，就说明了它的代价，就是把并发变成串行了（或许没有那么僵硬地让Ti+1一定在Ti后执行，但一定会操作编排顺序，避免所有冲突）。这个隔离级别最安全，但是性能也最差。


