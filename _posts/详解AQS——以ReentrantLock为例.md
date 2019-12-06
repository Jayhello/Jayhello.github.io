# 1 ReentrantLock

1. 废话不多说，同步最关键的就是获取锁，所以直接从`ReentrantLock`的`lock`方法看起。

   ```java
   public void lock() {
       sync.lock();
   }
   ```

   `ReentrantLock.lock()`调用了`sync`成员变量的`lock()`方法，那么`sync`变量是什么呢？

   ```java
   private final Sync sync;
   ```

   `sync`变量是`Sync`类的变量，`Sync`类是ReentrantLock类的一个内部**抽象**类，**`Sync`类继承于`AQS`类**，这就是`ReentrantLock`类与`AQS`类的直接联系：

   ```java
   abstract static class Sync extends AbstractQueuedSynchronizer {
       ...
   }
   ```

   我们还可以看到`ReentrantLock`类中还有两个内部类，一个叫`FairSync`，另一个叫`NonfairSync`，字面意思就是公平锁和非公平锁。

   ```java
   static final class FairSync extends Sync {
       ...
   }
   static final class NonfairSync extends Sync {
       ...
   }
   ```

   这两个类都继承自Sync类，看到这里大约可以明白，**`ReentrantLock`设为公平锁的时候`sync`变量就是`FairSync`类的实例，设为非公平锁时就时`NonfairSync`类的实例**。看一下`ReentrantLock`的构造函数验证一下。

   ```java
   public ReentrantLock() {
       // 默认情况下使用非公平锁
       sync = new NonfairSync();
   }
   public ReentrantLock(boolean fair) {
       // 设置了参数就用对应的锁
       sync = fair ? new FairSync() : new NonfairSync();
   }
   ```

   

2. 我们再来看一下`AQS`类的声明

   ```java
   public abstract class AbstractQueuedSynchronizer
       extends AbstractOwnableSynchronizer
       implements java.io.Serializable {
       ...
   }
   ```

   可以看到**`AQS`类是一个抽象类**，也就是说它是专门用来给别的类继承的。

3. 小结

   到这里可以小结一下`ReentrantLock`的基本结构：

   - `ReentrantLock`类中有一个`Sync`类继承于`AQS`类，又有两个类`FairSync`类、`NonfairSync`继承自`Sync`类。
   - `ReentrantLock`类中有一个`Sync`类的变量，当使用公平锁时就指向`FairSync`类的实例，使用非公平锁时就使用`NonfairSync`类的实例。
   - `ReentranLock`调用`lock`方法时就是调用`Sync`类的`lock`方法，而`Sync`类的`lock`方法实现依赖于`AQS`类，具体的实现下面会讨论。
   - 这就是`ReentrantLock`类与`AQS`类的关联，也是`AQS`类的使用方法。



# 2 获取锁

1. 还是从`Sync`类的`lock`方法看起。

   当使用非公平锁时：

   ```java
           final void lock() {
               // 直接尝试获取锁，compareAndSetState(0, 1)是AQS类中的方法，意思就是如果state为0，就把它设为1
               if (compareAndSetState(0, 1))
                   // 将占有排他锁锁的线程设为当前线程，这个也是AQS中的方法
                   setExclusiveOwnerThread(Thread.currentThread());
               else
                   // 否则就调用acquire方法，这是一个关键方法，下面再解释。
                   acquire(1);
           }
   ```

   `compareAndSetState`方法和`setExclusiveOwnerThread`方法的源码如下，都是一个原子操作：

   ```java
       protected final boolean compareAndSetState(int expect, int update) {
           return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
       }
   
       protected final void setExclusiveOwnerThread(Thread thread) {
           exclusiveOwnerThread = thread;
       }
   ```

   当使用公平锁时：

   ```java
           final void lock() {
               // 直接调用acquire方法
               acquire(1);
           }
   ```

   从上面可以看出，公平锁和非公平锁的lock()方法区别就是：

   - 公平锁的`lock()`方法被调用后就直接调用`AQS`类的`acquire()`方法。
   - 非公平锁`lock()`方法一被调用就马上去”抢“一次锁，如果没”抢“到才乖乖调用`acquire()`方法。
   - 可以猜测`acquire()`方法是要排队的。

2. `state`变量

   上面提到了一个`compareAndSetState`方法，修改`state`变量，`state`是`AQS`类中的一个变量，它记录了锁的状态，如果`state`为0表示锁没有被任何线程占有。当有线程请求锁的时候，可以带一个`acquire`参数，表示线程请求的资源数。如果请求成功，就将`acquire`加到`state`上，表示有线程占有了锁，并且持有了`state`个资源。

3. `acquire()`方法

   `acquire()`方法是AQS类中的方法，作用就是请求锁，源码如下：

   ```java
    	public final void acquire(int arg) {
           if (!tryAcquire(arg) &&
               acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
           {
               selfInterrupt();
           }
       }
   ```

   只有一个`if`语句，里面有一大串，我们先一步一步的解释，源码在下面分析：

   - 首先调用`tryAcquire(arg)`，也就是尝试占有锁，如果占有成功则直接退出。
   - 如果占有失败，调用`addWaiter(Node.EXCLUSIVE), arg)`以排他锁方式将当前线程加入阻塞队列。
   - 调用`acquireQueued()`方法，在队列中等待获取锁，直到成功获取到锁才返回，如果在等待过程中线程被中断过就返回`true`，否则返回`false`。
   - 如果在等待过程中线程被中断过，就再将该线程设为中断状态。这是因为在等待过程中，线程只知道傻等，别人告诉他”你被中断了“它是不响应的，要等待过程结束后再告诉它”你被中断了“。

4. `tryAcquire()`方法

   `tryAcquire()`是AQS类中的方法，但是AQS类不提供实现，需要靠子类来提供实现，如果调用了AQS类中的`tryAcquire()`方法将会抛出异常，源码如下：

   ```java
       protected boolean tryAcquire(int arg) {
           throw new UnsupportedOperationException();
       }
   ```

   那么这个方法必须要靠子类实现，为什么不把它设为抽象方法呢？这是因为AQS类提供了共享锁和排他锁两种形式，要实现排他锁的子类只需要实现`tryAcquire`、`tryRelease`方法，要实现共享锁的子类只需要实现`tryAcquireShared`、`tryReleaseShared`方法。如果将这些方法定义为抽象方法，那么实现排他锁的子类也要实现`tryAcquireShared`、`tryReleaseShared`方法，这是没必要的。所以用抛出异常的方式来禁止调用AQS类中的`tryAcquire`等方法。

   既然`tryAcquire()`方法需要由子类来实现，那就来看一下`FairSync`类和`NonfairSync`类是怎么实现`tryAcquire`的吧

   - `FairSync`类的`tryAcquire`

     ```java
         protected final boolean tryAcquire(int acquires) {
             	// 获取当前线程
                 final Thread current = Thread.currentThread();
             	// 获得锁状态
                 int c = getState();
             	// 如果锁状态为0，也就是没有被其他线程占有
                 if (c == 0) {
                     // 首先看有没有其他线程在排队，如果有就返回false，如果没有就进行下一步
                     // 使用CAS改变锁状态
                     // 将持有排他锁的线程设为当前线程
                     // 返回true
                     if (!hasQueuedPredecessors() &&
                         compareAndSetState(0, acquires)) {
                         setExclusiveOwnerThread(current);
                         return true;
                     }
                 }
             	// 如果锁状态不为0，但是持有排他锁锁的线程是当前线程
                 else if (current == getExclusiveOwnerThread()) {
                     // 表示重入锁，当前线程继续获取资源
                     int nextc = c + acquires;
                     // 如果nextc<0，表示state溢出了int的最大值，抛出错误
                     if (nextc < 0)
                         throw new Error("Maximum lock count exceeded");
                     // 如果state没有溢出，就设为新的值
                     setState(nextc);
                     // 返回true
                     return true;
                 }
                 return false;
             }
         }
     ```

   - `NonfairSync`的`tryAcquire`

     ```java
             protected final boolean tryAcquire(int acquires) {
                 return nonfairTryAcquire(acquires);
             }
     
             final boolean nonfairTryAcquire(int acquires) {
                 // 获取当前线程id
                 final Thread current = Thread.currentThread();
                 // 获取状态
                 int c = getState();
                 // 如果锁没有被占有
                 if (c == 0) {
                     // 使用CAS将锁状态设为acquire
                     if (compareAndSetState(0, acquires)) {
                         // 将持有排他锁的线程设为当前线程
                         setExclusiveOwnerThread(current);
                         return true;
                     }
                 }
                 // 重入锁逻辑，与公平锁相同。
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

   - 可见，公平锁的`tryAcquire`和非公平锁的`tryAcquire`区别就在于：**公平锁如果发现锁没有被占有，会看一下有没有其他线程在排队，如果没有线程在排队，才会用CAS去占有锁**，而非公平锁不会。也就是说公平锁会”文明“一点，而非公平锁更”自私“一些。

5. `addWaiter`方法

   顾名思义，增加一个等待者，也就是将线程加入等待队列。

   ```java
       private Node addWaiter(Node mode) {
           // 创建一个线程节点，使用当前线程初始化，mode表示锁的模式（排他锁还是共享锁）
           Node node = new Node(Thread.currentThread(), mode);
           // 获取等待队列的尾节点
           Node pred = tail;
           // 如果尾节点不为空
           if (pred != null) {
               node.prev = pred;
               // 使用cas将尾节点设为当前节点
               if (compareAndSetTail(pred, node)) {
                   pred.next = node;
                   return node;
               }
           }
           // 如果尾节点为null，或者CAS失败的话，就调用enq(node)方法
           enq(node);
           return node;
       }
   ```

   ```java
       private Node enq(final Node node) {
           // 死循环，直到当前节点放置成功为止
           for (;;) {
               Node t = tail;
               // 如果等待队列为空，就用CAS将头节点设为一个空节点（这个空节点应该是哨兵，这里使用了惰性初始化）
               if (t == null) { // Must initialize
                   if (compareAndSetHead(new Node()))
                       tail = head;
               } else {
                   // 否则就用CAS将当前节点设为尾节点
                   node.prev = t;
                   if (compareAndSetTail(t, node)) {
                       t.next = node;
                       return t;
                   }
               }
           }
       }
   ```

6. `acquireQueued`方法

   这个方法就是等待获取锁，直到成功为止，返回值为等待过程中线程是否被中断过。

   ```java
       final boolean acquireQueued(final Node node, int arg) {
           boolean failed = true;
           try {
               boolean interrupted = false;
               // 死循环，直到获取成功为止
               for (;;) {
                   // 获取当前线程节点的前置节点
                   final Node p = node.predecessor();
                   // 如果前置节点是head，也就是说当前节点已经在队列的第二个了
                   // 使用tryAcquire尝试获取锁
                   if (p == head && tryAcquire(arg)) {
                       // 如果获取成功，就将当前节点设为头节点
                       setHead(node);
                       // 将之前头节点的next引用置为空，也就是减少了当前节点的一个引用
                       p.next = null; // help GC
                       failed = false;
                       return interrupted;
                   }
                   // shouldParkAfterFailedAcquire检查当前节点是否需要挂起，唤醒挂起的条件是被调用unpark(),或者被中断，如果不需要就继续循环
                   // 如果需要挂起，则挂起线程，并检查线程是否中断
                   // 如果线程被中断，清除线程的中断标记位，将返回值设为true
                   if (shouldParkAfterFailedAcquire(p, node) &&
                       parkAndCheckInterrupt())
                       interrupted = true;
               }
           } finally {
               // 如果失败了，就取消获取锁。
               // 这个地方我比较费解，这个函数执行完只有三种情况：
               // 1. 如果函数成功返回了，此处failed必为false，此处代码不会被调用
               // 2. 如果函数没有返回，那就一直在循环中，也不会调用此处
               // 3. 还有一种情况是发生了异常，那么此处有可能会被调用，但是函数就没有返回值了。
               // 4. 如果只有发生异常的情况下此处才会被调用，那么为什么不放在catch块里
               if (failed)
                   cancelAcquire(node);
           }
       }
   ```

7. `shouldParkAfterFailedAcquire`方法

   ```java
       private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
           // 取前一个节点的waitStatus
           int ws = pred.waitStatus;
           // 一个节点的waitStatus==SIGNAL表示当前节点的下一个节点应该等待
           if (ws == Node.SIGNAL)
               return true;
           // waitStatus>0表示当前节点已经被取消
           if (ws > 0) {
               // 那就一直往前找，并释放这些被取消的节点，直到找到没有被释放的那个节点为止
               do {
                   node.prev = pred = pred.prev;
               } while (pred.waitStatus > 0);// 这里没有判断pred != null，是因为一直往前找会找到head节点，而head是当前持有锁的线程，一定不会被取消了
               pred.next = node;
           } else {
               // 否则，用CAS将前一个节点的waitStatus设为SIGNAL
               compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
           }
           // 返回false
           return false;
       }
   ```



# 3 释放锁

上面讲到了，`shouldParkAfterFailedAcquire`检查当前节点是否需要挂起，唤醒挂起的条件是被调用`unpark()`,或者被中断。那么什么时候挂起线程被唤醒呢，因该是在其他线程释放锁之后，所以我们先从`ReentrantLock`类的`unlock()`方法看起。

```java
    public void unlock() {
        sync.release(1);
    }
```

1. `release()`方法

   这个方法是继承自`AQS`类的：

   ```java
    public final boolean release(int arg) {
           // 首先tryRelease
           if (tryRelease(arg)) {
               Node h = head;
               if (h != null && h.waitStatus != 0)
                   // 从头节点开始，唤醒下一个能被唤醒的节点
                   unparkSuccessor(h);
               return true;
           }
           return false;
       }
   ```

2. `tryRelease()`方法

   这个方法实现在`Sync`类中，也就是说，公平锁和非公平锁的释放用的是同一个`tryRelease`。

   ```java
           protected final boolean tryRelease(int releases) {
               // 减少state
               int c = getState() - releases;
               // 如果当前线程不是占有排他锁的线程，抛出异常
               if (Thread.currentThread() != getExclusiveOwnerThread())
                   throw new IllegalMonitorStateException();
               boolean free = false;
               // 如果state==0，表示当前线程已经完全释放锁
               if (c == 0) {
                   free = true;
                   // 将占有排他锁的线程设为null
                   setExclusiveOwnerThread(null);
               }
               // 更新state
               setState(c);
               // 返回释放成功
               return free;
           }
   ```

3. `unparkSuccessor`方法

   ```java
       private void unparkSuccessor(Node node) {
           // 这个函数的作用是唤醒当前节点后面找到一个能被唤醒的线程。
           int ws = node.waitStatus;
           // 如果当前线程的waitStatus < 0，就将它设为0，确保它后面的节点能够被唤醒
           if (ws < 0)
               compareAndSetWaitStatus(node, ws, 0);
   
           Node s = node.next;
           
           if (s == null || s.waitStatus > 0) {
               // 如果下一个节点为null，或者下一个节点被取消了
               s = null;
               for (Node t = tail; t != null && t != node; t = t.prev)
                   // 就从后往前找到第一个waitStatus<=0的节点，唤醒这个节点
                   if (t.waitStatus <= 0)
                       s = t;
           }
           if (s != null)
               LockSupport.unpark(s.thread);
       }
   ```

   

