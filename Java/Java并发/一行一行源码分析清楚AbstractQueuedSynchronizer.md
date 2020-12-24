#### 本文转自 [一行一行源码分析清楚AbstractQueuedSynchronizer](https://javadoop.com/post/AbstractQueuedSynchronizer)

在分析 Java 并发包 java.util.concurrent 源码的时候，少不了需要了解 AbstractQueuedSynchronizer（以下简写AQS）这个抽象类，因为它是 Java 并发包的基础工具类，是实现 ReentrantLock、CountDownLatch、Semaphore、FutureTask 等类的基础。

Google 一下 AbstractQueuedSynchronizer，我们可以找到很多关于 AQS 的介绍，但是很多都没有介绍清楚，因为大部分文章没有把其中的一些关键的细节说清楚。

本文将从 ReentrantLock 的公平锁源码出发，分析下 AbstractQueuedSynchronizer 这个类是怎么工作的，希望能给大家提供一些简单的帮助。

## AQS
先来看看 AQS 有哪些属性，搞清楚这些基本就知道 AQS 是什么套路了，毕竟可以猜嘛！

```
// 头结点，你直接把它当做 当前持有锁的线程 可能是最好理解的
private transient volatile Node head;

// 阻塞的尾节点，每个新的节点进来，都插入到最后，也就形成了一个链表
private transient volatile Node tail;

// 这个是最重要的，代表当前锁的状态，0代表没有被占用，大于 0 代表有线程持有当前锁
// 这个值可以大于 1，是因为锁可以重入，每次重入都加上 1
private volatile int state;

// 代表当前持有独占锁的线程，举个最重要的使用例子，因为锁可以重入
// reentrantLock.lock()可以嵌套调用多次，所以每次用这个来判断当前线程是否已经拥有了锁
// if (currentThread == getExclusiveOwnerThread()) {state++}
private transient Thread exclusiveOwnerThread; // 继承自AbstractOwnableSynchronizer
```

怎么样，看样子应该是很简单的吧，毕竟也就四个属性啊。

AbstractQueuedSynchronizer 的等待队列示意如下所示，注意了，之后分析过程中所说的 queue，也就是**阻塞队列不包含 head，不包含 head，不包含 head**。

