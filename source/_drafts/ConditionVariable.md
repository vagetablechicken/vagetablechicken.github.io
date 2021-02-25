---
title: ConditionVariable
tags:
---

## Condition Variable & Mutex

Why CondVar needs mutex?
我觉得这个问题主要是不理解条件变量CondVar为什么要出现。
CondVar这个东西虽然很底层，但它也不是magic，而且这么难懂，很明显它是一个被抽象了的东西。
那么它为什么要出现？
> https://web.stanford.edu/~ouster/cgi-bin/cs140-spring14/lecture.php?topic=locks
首先，假设现在只有mutex，或者更抽象的说，只有一个lock，没有其他东西，我们要实现两个线程，一个生产，一个消费怎么做？
```
producer:
    lock
    add one to queue
    unlock
```
```
consumer:
    lock
    get one from queue
    unlock
```
可以看到，这么做，可以保证线程安全。但是很明显，这个逻辑有很多缺点，比如，producer和consumer处理一次的耗时不一样，甚至每次处理时长都不一样，producer快了，queue就爆炸了，consumer快了，就会想从空队列里拿东西。更进一步，consumer如果get空，很快进入下一轮，又去抢锁，lock/unlock频率很高，很浪费，而且还可能阻碍producer抢到锁。

那么改进思路，必然是需要一个有限的队列。

但是consumer发现queue empty，它应该怎么做？它当然应该放弃锁，期待producer拿了锁往里面装点东西。但是consumer总得知道queue not empty，它必须重复查询，显然可以写为while，如下所示。
```
consumer1:
    lock
    while queue is empty:
        unlock
        # sleep?
        lock
    get one from queue
    unlock
```
但是这个consumer1的while queue is empty并没有高明到哪里去，如果不停一停，它和上一个版本consumer没啥区别。但sleep不是多么棒的主意，sleep多久呢？又决定不下来了。

而如果有谁给我们提供一个很棒的机制，做到了别人signal才会唤醒sleep的consumer，那不就完美了？
于是就有了CondVar，我们可以将CondVar简单的理解为：
```
CondVar:
    unlock
    wait signal
    lock
```
`wait signal`当然不是一句话这么简单，但反正无论多复杂，都被CondVar包含了，你只要使用就可以了。

于是，producer和consumer都要变了：
```
producer2:
    lock
    add one to queue
    CondVar.notify()
    unlock

consumer2:
    lock
    while queue is empty:
        CondVar.wait(&lock)
    get one from queue
    unlock
```
可以看到，这个lock后while+wait的写法，就是因为CondVar封装了细节，但是却必须让你传入参数lock，不然它无法使用这个lock。

这就是'Why CondVar needs mutex?'的答案。
## Posix Cond

wait的使用没有什么疑问。signal thread在改变condition时是必须加锁的。
因为不加锁，可能会出现这种顺序：
```
Process A                             Process B

pthread_mutex_lock(&mutex);
while (condition == FALSE)

                                      condition = TRUE;
                                      pthread_cond_signal(&cond);

pthread_cond_wait(&cond, &mutex);
```
from https://stackoverflow.com/a/4567919

可以看到，如果不加锁，就无法保证condition change在wait thread**真的wait**的时期发生。就可能漏掉signal。
而保证condition change加锁，Process A，要不第一次while判断就能check到condition，要不进入cond wait，会等到下一次加锁，又走到while判断。换个角度也可以说是，while判断这一句是加了锁的，按并发的基本逻辑，condition change自然也是该加锁的。

一句话，只要你能保证condition change在锁期间，signal在锁内还是解锁后都是不会漏signal的。放心写。


signal的位置需要思考。

```
lock
cond_signal
unlock
```
这个是有上锁冲突的可能的。极端情况下，你signal了，wait的线程立马被唤醒，而你这边还没unlock，wait的线程就发现拿不了锁，它会怎么做？
>On a uni-processor this would often not be a
problem, but on a multi-processor the effect is liable to be that a potential writer is
awakened inside “Wait”, executes a few instructions, and then blocks trying to lock the
mutex—because it is still held by the terminating reader, executing concurrently. A few
microseconds later the terminating reader unlocks the mutex, allowing the writer to
continue. This has cost us two extra re-schedule operations, which is a significant
expense.
https://www.cs.utexas.edu/~dahlin/Classes/GradOS/papers/threads.pdf

所以就会等一下unlock，会多耗两个re-schedule operations，这又是啥？

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