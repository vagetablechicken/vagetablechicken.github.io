---
title: ConditionVariable
tags:
---

## Posix Cond

wait的使用没有什么疑问。
signal的位置需要思考。

```
lock
cond_signal
unlock
```
这个是有上锁冲突的可能的。极端情况下，你signal了，wait的线程立马被唤醒，而你这边还没unlock，wait的线程就发现拿不了锁，它会怎么做？
Posix是明确允许这么做的，上锁冲突不是必须避免的事情，事实上，为了保证是wait的线程先拿到锁（比其他直接想拿锁的线程优先级高），你还必须把signal放在锁区间里。

linux里上锁冲突代价并不离谱？要看使用的posix具体实现。
等待线程从内核中唤醒（由于cond_signal)然后又回到内核空间（因为cond_wait返回后会有原子加锁的 行为），所以一来一回会有性能的问题。但是在LinuxThreads或者NPTL里面，就不会有这个问题，因为在Linux 线程中，有两个队列，分别是cond_wait队列和mutex_lock队列， cond_signal只是让线程从cond_wait队列移到mutex_lock队列，而不用返回到用户空间，不会有性能的损耗。
所以在Linux中推荐使用这种模式。

```
lock
unlock
cond_signal
```
这个是可以避免上锁冲突的。