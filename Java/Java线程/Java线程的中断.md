## Java线程的中断

转载自  [Java线程的中断](https://zhuanlan.zhihu.com/p/67192849)

## 引言
### 一、中断相关的方法介绍
设计到中断的线程基础方法有三个：interrupt()、isInterrupted()、interrupted()，它们都位于Thread类下。

- interrupt() 方法：对目标线程发送中断请求，看起源码会发现最终是调用了一个本地方法实现的线程中断。

- isInterrupted() 方法：返回目标线程是否中断的布尔值(通过本地方法实现)，且返回后会重置中断状态为未中断。

- interrupted() 方法： 该方法返回的是线程中断与否的布尔值(通过本地方法实现)，不会重置中断状态。

### 二、线程中断
线程中断可以按中断时线程状态分为两类，一类是运行时线程的中断，一类是阻塞或等待线程的中断。有中断时，运行时的线程会在某个取消点中断执行，而处于阻塞或者等待状态的线程大多会立即响应中断，比如 join、sleep 等方法，这些方法在抛出中断异常的错误后，会重置线程中断状态为未中断。但注意，**获取独占锁的阻塞状态与BIO的阻塞状态不会响应中断**。而在JUC包中有在加锁阻塞的过程中响应中断的方法，比如lockInterruptibly()。

#### 1、线程中断的目的是什么
有时是由于对于某种特定情况，我们知道当前线程无需继续执行下去，此时可以中断此线程；有时是遇到某些异常，需要中断线程。具体什么目的，还要看具体场景，但线程中断的需求已经摆在那里，肯定需要。

#### 2、要如何处理线程中断
通常的处理方式有两种，如果是业务层面的代码，则只需要做好中断线程之后的业务逻辑处理即可，而如果是偏底层功能的线程中断，则尽量将中断异常抛出（或者在catch中重新调用interrupt()来中断线程），以告知上层方法本线程的中断经历。

#### 3、JUC中对中断的处理举例
JUC中ReentrantLock常用的加锁方法是lock()，还有一个响应中断的加锁方法lockInterruptibly()。

lock()方法中的acquire(int arg)方法如下所示：

```
public final void acquire(int arg) {
      if (!tryAcquire(arg) &&
             acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
           selfInterrupt();
}
```

在acquireQueued中会对线程的中断状态做判断，如果中断了则返回true，进入selfInterrupt()方法，恢复线程的中断状态。但注意此处是在获取到锁之后再响应中断，在获取到锁之前不会做出响应。

```
static void selfInterrupt() {
         Thread.currentThread().interrupt();
 }
```

而看lockInterruptibly()方法：

```
public void lockInterruptibly() throws InterruptedException {
          sync.acquireInterruptibly(1);
      }
  
  public final void acquireInterruptibly(int arg)
              throws InterruptedException {
          if (Thread.interrupted())
              throw new InterruptedException();
          if (!tryAcquire(arg))
             doAcquireInterruptibly(arg);
     }
```

它会先查看中断状态，再获取锁。而如果在获取锁的过程中中断过，则会在doAcquireInterruptibly方法中抛出中断异常。

下面是我在本地模拟的lock阻塞中断：

```
package com.demo;

import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {

  public static void main(String[] args) throws InterruptedException {
    System.out.println("main start");
    Thread t1 = new Thread(new LockThreadDemo());
    Thread t2 = new Thread(new LockThreadDemo());
    t1.start();
    Thread.sleep(1000); // 确保thread1获取到了锁
    t2.start(); // 此时thread2处于获取锁的阻塞状态
    t2.interrupt();
    System.out.println("main end");
  }
}

class LockThreadDemo implements Runnable {

  public static ReentrantLock lock = new ReentrantLock();

  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName() + "runnable run");
    try {
      lock.lock();
      System.out.println(Thread.currentThread().getName() + "开始睡眠");
      Thread.sleep(5000);
      System.out.println(Thread.currentThread().getName() + "睡了5秒");
    } catch (Exception e) {
      System.out.println(Thread.currentThread().getName() + "runnable exception:" + e);
    } finally {
      lock.unlock();
    }
    System.out.println(Thread.currentThread().getName() + " over");
  }
}
```

执行结果为：

```
main start
Thread-0runnable run
Thread-0开始睡眠
main end
Thread-1runnable run
Thread-0睡了5秒
Thread-0 over
Thread-1开始睡眠
Thread-1runnable exception:java.lang.InterruptedException: sleep interrupted
Thread-1 over
```

可以看到中断了并没有对获取锁产生影响，最后是sleep方法响应的中断。

下面是我在本地模拟的lockInterruptibly()阻塞中断：

```
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockInterruptableDemo {

  public static void main(String[] args) throws InterruptedException {
    System.out.println("main start");
    Thread t1 = new Thread(new LockThreadInterruptableDemo());
    Thread t2 = new Thread(new LockThreadInterruptableDemo());
    t1.start();
    Thread.sleep(1000); // 确保thread1获取到了锁
    t2.start(); // 此时thread2处于获取锁的阻塞状态
    t2.interrupt();
    System.out.println("main end");
  }
}

class LockThreadInterruptableDemo implements Runnable {
  public static ReentrantLock lock = new ReentrantLock();

  @Override
  public void run() {
    System.out.println(Thread.currentThread().getName() + "runnable run");
    try {
      lock.lockInterruptibly();
      System.out.println(Thread.currentThread().getName() + "开始睡眠");
      Thread.sleep(5000);
      System.out.println(Thread.currentThread().getName() + "睡了5秒");
    } catch (Exception e) {
      System.out.println(Thread.currentThread().getName() + "runnable exception:" + e);
    } finally {
      try {
        lock.unlock();
      } catch (IllegalMonitorStateException e) {
        System.out.println("因线程" + Thread.currentThread().getName() + "提前中断导致未获取到锁");
      }
    }
    System.out.println(Thread.currentThread().getName() + " over");
  }
}
```

结果为：

```
main start
Thread-0runnable run
Thread-0开始睡眠
main end
Thread-1runnable run
Thread-1runnable exception:java.lang.InterruptedException
因线程Thread-1提前中断导致未获取到锁
Thread-1 over
Thread-0睡了5秒
Thread-0 over
```
