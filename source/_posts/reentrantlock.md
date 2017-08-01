---
title: ReentrantLock你真的了解吗
date: 2017-07-01 15:57:25
tags:
- JDK源码
- Lock
- ReentrantLock
categories:
- 高并发
---

你真的了解ReentrantLock吗？顾名思义，这是可重入锁的意思，废话不多说我们直接看源码：
## 结构组成
```
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;

    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
    ...
    }
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        
    }

    /**
     * Sync object for fair locks
     */
    static final class FairSync extends Sync {
    }
    
```
世人都知道这个是基于AQS实现的，AQS具体是什么，不是本文重点，后续解释；
1. ReentrantLock 有个抽象内部类，Sync 继承了AQS  
2. 有两个静态内部类，FairSync 用与公平锁，NonFairSync用与非公平锁.  

## 初始化
1. 默认初始化
```
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```
默认初始化生成了一个非公平锁；

## 加锁操作
```
 final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
```
compareAndSetState（）方法，设置当前的锁占用状态；实际上是设置了AQS的state的状态，由0设置为1，unsafe是native实现。（并发包里有好多工具类的实现，最后的落脚点在unsafe的native方法上）。  
- 如果设置成功，表示线程获初次取锁成功，然后设置当前的线程为锁的独占线程（setExclusiveOwnerThread（））。
```
    protected final boolean compareAndSetState(int expect, int update) {
        // See below for intrinsics setup to support this
        return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
    }
```
- 如果失败，表示已经有线程获得了锁，做可重入的处理。
acquire(1),此方法是AQS的final方法：
```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```
tryAcquire是抽象方法，在NonSync中overwrite了，这个源码上面已经提到了，NonSync中此方法调用了nonfairTryAcquire（），我们看一下做了什么操作：
```
/**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
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
getState()是获取state的数值，此值是个volatile变量（主要是保持内存可见性）,用来标记重入锁的重入次数
```
    /**
     * The synchronization state.
     */
    private volatile int state;
```
- 如果c==0 表示初次获取锁，此时set  state=1 ,然后设置独占锁的线程为当前线程；
- 如果c !=0 表示锁已被获取，判读是否加锁操作是已经占领锁的线程发起的，如果是，就对state+1 操作，重入次数+1； （nextc<0 属于加锁次数达到int最大值溢出所致）
- 如果非当前线程尝试加锁，就返回false了

回到上述的acquire方法，判读条件：
```
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
      selfInterrupt();
    
    static void selfInterrupt() {
        Thread.currentThread().interrupt();
    }
```
不满足条件重入锁的条件，做入队操作（这里的入队实际上是入链表操作）,入队成功，调用thread的中断方法（具体中断机制这里不描述，感兴趣的可以自行学习），继续深入源码，入队部分：
```
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
第二个if： 获取锁失败，park 并检查中断标志位，然后设置中断标志位为true；
```
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```
关键的线程阻塞代码；park操作
```
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```
我们看下park方法源码：
```
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```
调用了unsafe的park方法：最终把线程交给系统（linux）内核进行阻塞；

落实到unsafe方法的时候，好多 CAS（compare and set），太常见了，O(∩_∩)O~，
```
    /**
     * CAS waitStatus field of a node.
     */
    private static final boolean compareAndSetWaitStatus(Node node,
                                                         int expect,
                                                         int update) {
        return unsafe.compareAndSwapInt(node, waitStatusOffset,
                                        expect, update);
    }
```

## 解锁部分
此部分略过...

## Lock VS Synchronized
AbstractQueuedSynchronizer通过构造一个基于阻塞的CLH队列容纳所有的阻塞线程，而对该队列的操作均通过Lock-Free（CAS）操作，但对已经获得锁的线程而言，ReentrantLock实现了偏向锁的功能。
synchronized的底层也是一个基于CAS操作的等待队列，但JVM实现的更精细，把等待队列分为ContentionList和EntryList，目的是为了降低线程的出列速度；当然也实现了偏向锁，从数据结构来说二者设计没有本质区别。但synchronized还实现了自旋锁，并针对不同的系统和硬件体系进行了优化，而Lock则完全依靠系统阻塞挂起等待线程。
当然Lock比synchronized更适合在应用层扩展，可以继承AbstractQueuedSynchronizer定义各种实现，比如实现读写锁（ReadWriteLock），公平或不公平锁；同时，Lock对应的Condition也比wait/notify要方便的多、灵活的多。

> 参考链接：  
http://blog.csdn.net/chen77716/article/details/6641477
http://www.cnblogs.com/timlearn/p/4008783.html
