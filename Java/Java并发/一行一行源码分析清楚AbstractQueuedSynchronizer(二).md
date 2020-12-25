#### 本文转自 [一行一行源码分析清楚AbstractQueuedSynchronizer](https://javadoop.com/post/AbstractQueuedSynchronizer-2)

本文关注以下几点内容：

- 深入理解 ReentrantLock 公平锁和非公平锁的区别

- 深入分析 AbstractQueuedSynchronizer 中的 ConditionObject

- 深入理解 Java 线程中断和 InterruptedException 异常


## 公平锁和非公平锁
ReentrantLock 默认采用非公平锁，除非你在构造方法中传入参数 true 。

```
public ReentrantLock() {
    // 默认非公平锁
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

公平锁的 lock 方法：

```
static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }

    // AbstractQueuedSynchronizer.acquire(int arg)s
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 1. 和非公平锁相比，这里多了一个判断：是否有线程在等待
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

非公平锁的 lock 方法：

```
static final class NonfairSync extends Sync {
    final void lock() {
        // 2. 和公平锁相比，这里会直接先进行一次CAS，成功就返回了
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
    // AbstractQueuedSynchronizer.acquire(int arg)
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}

/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 这里没有对阻塞队列进行判断
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

总结：公平锁和非公平锁只有两处不同：

- 非公平锁在调用 lock 后，首先就会调用 CAS 进行一次抢锁，如果这个时候恰巧锁没有被占用，那么直接就获取到锁返回了。

- 非公平锁在 CAS 失败后，和公平锁一样都会进入到 tryAcquire 方法，在 tryAcquire 方法中，如果发现锁这个时候被释放了（state == 0），非公平锁会直接 CAS 抢锁，但是公平锁会判断等待队列是否有线程处于等待状态，如果有则不去抢锁，乖乖排到后面。

公平锁和非公平锁就这两点区别，如果这两次 CAS 都不成功，那么后面非公平锁和公平锁是一样的，都要进入到阻塞队列等待唤醒。

相对来说，非公平锁会有更好的性能，因为它的吞吐量比较大。当然，非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态。


## Condition
Tips: 这里重申一下，要看懂这个，必须要先看懂上一篇关于 [AbstractQueuedSynchronizer](https://github.com/xinput123/about-me/blob/main/Java/Java%E5%B9%B6%E5%8F%91/%E4%B8%80%E8%A1%8C%E4%B8%80%E8%A1%8C%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E6%B8%85%E6%A5%9AAbstractQueuedSynchronizer(%E4%B8%80).md) 的介绍，或者你已经有相关的知识了，否则这节肯定是看不懂的。


我们先来看看 Condition 的使用场景，Condition 经常可以用在生产者-消费者的场景中，请看 Doug Lea 给出的这个例子：

> - 1、我们可以看到，在使用 condition 时，必须先持有相应的锁。这个和 Object 类中的方法有相似的语义，需要先持有某个对象的监视器锁才可以执行 wait(), notify() 或 notifyAll() 方法。

> - 2、ArrayBlockingQueue 采用这种方式实现了生产者-消费者，所以请只把这个例子当做学习例子，实际生产中可以直接使用 ArrayBlockingQueue







