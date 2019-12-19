# 1 AQS类简介

1. 首先`AQS`全称叫`AbstractQueuedSynchronizer`，意思就是**抽象的队列同步器**。它是一个抽象类，需要实现共享锁或者排他锁的类可以通过继承这个类，并覆盖其中的`tryAcquire(int)-tryRelease(int)`或者`tryAcquireShared(int)-tryReleaseShared(int)`中的一组。

2. `state`变量

   `state`是`AQS`类中的一个`volatile int`变量，它记录了锁的状态。如果`state`为0表示锁没有被任何线程占有。当有线程请求锁的时候，可以带一个`acquire`参数，表示线程请求的资源数。如果请求成功，就将`acquire`加到`state`上。

   `AQS`中提供了三种方法来修改此变量：

   - getState()
   - setState()
   - compareAndSetState()

3. CLG队列

   `AQS`内维护了一个等待队列，实际上是一个双向链表，链表的每个节点表示一个线程，链表的头节点就是当前持有锁的线程。

   ![img](../images/%E8%AF%A6%E8%A7%A3AQS%E2%80%94%E2%80%94%E4%BB%A5ReentrantLock%E4%B8%BA%E4%BE%8B.assests/721070-20170504110246211-10684485.png)

4. 自定义同步器

   `AQS`类中有以下几个方法属于模板方法，需要子类覆盖才能正确访问。

   - `isHeldExclusively()`：该线程是否正在独占资源。只有用到condition才需要去实现它。
   - `tryAcquire(int)`：独占方式。尝试获取资源，成功则返回true，失败则返回false。
   - `tryRelease(int)`：独占方式。尝试释放资源，成功则返回true，失败则返回false。
   - `tryAcquireShared(int)`：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
   - `tryReleaseShared(int)`：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回false。

   要实现自定义同步器，首先要继承`AQS`类，然后根据要实现的是排它锁还是共享锁，选择实现`tryAcquire(int)-tryRelease(int)`，或者`tryAcquireShared(int)-tryReleaseShared(int)`中的一组。





# 2 ReentrantLock

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

   我们还可以看到`ReentrantLock`类中还有两个内部类，一个叫`FairSync`，另一个叫`NonfairSync`，字面意思就是**公平锁**和**非公平锁**。

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
   - `ReentrantLock`类中有一个`Sync`类的变量，当使用公平锁时就指向`FairSync`类的实例，使用非公平锁时就指向`NonfairSync`类的实例。
   - `ReentranLock`调用`lock`方法时就是调用`Sync`类的`lock`方法，而`Sync`类的`lock`方法实现依赖于`AQS`类，具体的实现下面会讨论。
   - 这就是`ReentrantLock`类与`AQS`类的关联，也是`AQS`类的使用方法。





## 2.1 获取锁

1. 还是从`Sync`类的`lock`方法看起。

   当使用**非公平锁**时：

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

   当使用**公平锁**时：

   ```java
           final void lock() {
               // 直接调用acquire方法
               acquire(1);
           }
   ```

   从上面可以看出，公平锁和非公平锁的`lock()`方法区别就是：

   - 公平锁的`lock()`方法被调用后就直接调用`AQS`类的`acquire()`方法。
   - 非公平锁`lock()`方法一被调用就马上去”抢“一次锁，如果没”抢“到才乖乖调用`acquire()`方法。
   - 可以猜测`acquire()`方法是要排队的。

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

   这个方法既然必须要靠子类实现，那么为什么不把它设为抽象方法呢？这是因为AQS类提供了共享锁和排他锁两种形式，要实现排他锁的子类只需要实现`tryAcquire`、`tryRelease`方法，要实现共享锁的子类只需要实现`tryAcquireShared`、`tryReleaseShared`方法。如果将这些方法都定义为抽象方法，那么实现排他锁的子类也要实现`tryAcquireShared`、`tryReleaseShared`方法，这会给程序员带来额外的负担。所以用抛出异常的方式来禁止调用AQS类中的`tryAcquire`等方法。这实际上使用了**模板方法设计模式**。

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
                     // 返回true，表示成功
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
                     // 不管有没有其他线程排队，使用CAS将锁状态设为acquire
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

4. `addWaiter`方法

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
                   // 着眼，运行到这里只有node.prev = pred设置成功了，pred.next = node还没有设置，如果此线程运行到这里被系统调度交出了运行权，或者因为某种原因死掉了，那么双向链表就只有从后往前是通的，这就是为什么后面在遍历链表时都是从后往前遍历，后面会看到。
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
           // 自旋，直到当前节点放置成功为止
           for (;;) {
               Node t = tail;
               // 如果等待队列为空，就用CAS将头节点设为一个空节点（这个空节点应该是哨兵，这里使用了惰性初始化，就是说在初始化锁对象时没有创建链表，直到第一次使用锁时才创建）
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
               // 自旋，直到获取成功为止
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
       // 这个函数的字面意思是判断自己能否park了，作用就是将前面的节点的waitStatus设为SIGNAL（-1），告诉它如果用完锁了就把自己叫醒，返回是否设置成功
   	private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
           // 取前一个节点的waitStatus
           int ws = pred.waitStatus;
           // 如果前一个结点已经知道这事了，那就不要烦他了，直接返回true
           if (ws == Node.SIGNAL)
               return true;
           
           // ws>0表示前面的节点已经被取消，排在前面的老哥撂挑子不干了
           if (ws > 0) {
               // 那就一直往前找，跳过并释放这些被取消的节点，直到找到没有被释放的那个节点为止
               do {
                   node.prev = pred = pred.prev;
               } while (pred.waitStatus > 0);// 这里没有判断pred != null，是因为一直往前找会找到head节点，而head要么是当前持有锁的线程，要么是默认初始化的哨兵节点，一定不会是被取消了的节点
               pred.next = node;
           } else {
               // 前面的节点没有被取消，用CAS将前一个节点的waitStatus设为SIGNAL，告诉他一会叫我
               compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
           }
           // 这里会返回false，当前线程不会被挂起，但是没关系，这个函数是在自旋中调用的，下一次自旋还会进来，那时候就可能会返回true了
           return false;
       }
   ```
   
7. `parkAndCheckInterrupt()`方法

   ```java
   private final boolean parkAndCheckInterrupt() {
       LockSupport.park(this);//调用park()使线程进入waiting状态
       
       // 这里线程被park()进入waiting状态，只有被unpark()或者被中断才能唤醒
       
       return Thread.interrupted();//执行到这里说明被唤醒了，返回自己是不是被中断唤醒的。
    }
   ```

   



## 2.2 释放锁

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
               // 返回释放结果
               return free;
           }
   ```

3. `unparkSuccessor`方法

   ```java
       private void unparkSuccessor(Node node) {
           // 这个函数的作用是唤醒当前节点后面找到一个能被唤醒的线程。
           int ws = node.waitStatus;
           // 如果当前线程的waitStatus < 0，就将它设为0，使它后面的节点能够被唤醒，因为如果waitStatus<0表示后续节点应该等待。
           if (ws < 0)
               compareAndSetWaitStatus(node, ws, 0);
   
           Node s = node.next;
           
           if (s == null || s.waitStatus > 0) {
               // 如果下一个节点为null（为null的可能原因在addWaiter方法中提到了），或者下一个节点被取消了
               s = null;
               for (Node t = tail; t != null && t != node; t = t.prev)
                   // 就从后往前找到第一个waitStatus<=0的节点，唤醒这个节点，为什么从后往前找，在addWaiter方法中有提到
                   if (t.waitStatus <= 0)
                       s = t;
           }
           if (s != null)
               LockSupport.unpark(s.thread);
       }
   ```




# 3 ReentrantReadWriteLock

`ReentrantReadWriteLock`意思就是可重入的读写锁，一般用在对资源的读写时。这个类实际上维护了两把锁，一个是读锁，一个是写锁。我们知道，多个线程同时读一个资源一般不会出错，但是多个线程同时写或者一边写一边读就可能会出错。因此读锁是共享的，而写锁是排它的。多线程并发访问统一资源时，存在以下四种情况：

| 当前持有锁的线程A                    | 另一线程B    | 结果      |
| ------------------------------------ | ------------ | --------- |
| 持有读锁（也可以是多个线程同时持有） | 尝试获取读锁 | B获取成功 |
| 持有读锁（也可以是多个线程同时持有） | 尝试获取写锁 | B获取失败 |
| 持有写锁                             | 尝试获取读锁 | B获取失败 |
| 持有写锁                             | 尝试获取写锁 | B获取失败 |

了解了这一基本的逻辑之后，我们来看一下`ReentrantReadWriteLock`的源码。

## 3.1 WriteLock

在`ReentrantReadWriteLock`中由两个内部类，一个是`ReadLock`，实现读锁，另一个是`WriteLock`实现写锁。

由于`ReentrantReadWriteLock`存在读锁和写锁两种，分别都有获取锁方法，在介绍`ReentrantLock`的时候我们了解了排他锁，因此为方便起见，这里也先介绍同为排他锁的写锁。

### 3.1.1 获取锁

1. `lock()`方法

   同样，先从`lock()`方法看起：

   ```java
           public void lock() {
               // 调用acquire方法
               sync.acquire(1);
           }
   ```

   ```java
       // 这个是AQS类中的acquire方法，上面解释过，就不解释了
   	public final void acquire(int arg) {
           if (!tryAcquire(arg) &&
               acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
               selfInterrupt();
       }
   ```

   

2. `tryAcquire()`方法

   `lock()`方法的关键在于`tryAcquire()`方法，它是由子类自己实现的。在`ReentrantReadWriteLock`中，`tryAcquire()`并没有分公平锁和非公平锁分别实现，而是直接在`Sync`类中实现：

   ```java
           protected final boolean tryAcquire(int acquires) {
               /*
                * 源码中这里有一段英文注释解释了这个方法的流程，如果能看懂的话理解起来会快很多
                * 1. 如果读锁计数非0，或者写锁计数非0且持有锁的线程又不是本线程，则返回false
                * 2. 如果锁的计数将要溢出，返回false
                * 3. 如果走到这里，则此线程有可能有资格获取锁，条件是它要么是一个重入进来的锁，要么是在等待队列允许它获得锁。如果满足条件，则更新state，然后将排它锁的线程设为自己。
                */
               
               Thread current = Thread.currentThread();
               // 获得当前state
               int c = getState();
               // 获取写锁的计数值w，如果不理解什么是写锁计数，先看一下下面的解释。
               int w = exclusiveCount(c);
               // c!=0表示有线程持有锁，既然有线程持有锁那么只有一种情况才有可能成功，那就是当前线程是重入进来的排他锁
               if (c != 0) {
                   // 如果写锁计数=0，说明有线程持有读锁——失败
                   // 如果写锁计数!=0，说明有线程持有写锁，判断是不是自己，如果不是——失败
                   if (w == 0 || current != getExclusiveOwnerThread())
                       return false;
                   // 如果写锁计数溢出——失败、抛出错误
                   if (w + exclusiveCount(acquires) > MAX_COUNT)
                       throw new Error("Maximum lock count exceeded");
                   //重入锁
                   setState(c + acquires);
                   return true;
               }
               
               // 到这里说明state=0
               // 首先调用writerShouldBlock()，查看获取写锁是否需要等待，这个方法是由FairSync类和NonfairSync类实现的，公平锁与非公平锁的区别就体现在这里。如果需要等待——失败。
               // 使用CAS尝试更新state
               if (writerShouldBlock() ||
                   !compareAndSetState(c, c + acquires))
                   return false;
               
               // 走到这里，说明成功获取了锁，将持有排他锁的线程设为自己并返回true
               setExclusiveOwnerThread(current);
               return true;
           }
   ```

3. 读锁计数与写锁计数

   在上面一段代码中提到了锁计数，具体是什么意思呢？原来，在`ReentrantReadWriteLock`中虽然维护了两种锁，但是两种锁公用`AQS`中的`state`变量。**读锁计数使用高16位，写锁计数使用低16位。**上面的`exclusiveCount(c)`就是将c的低16位取出来，同理还有一个`sharedCount(int c)`是将c的高16位取出来。

   ```java
           static final int SHARED_SHIFT   = 16;
           static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
           static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
           static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
   
           /** Returns the number of shared holds represented in count  */
           static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
           /** Returns the number of exclusive holds represented in count  */
           static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
   ```

   ![1576738298445](../images/%E8%AF%A6%E8%A7%A3AQS%E2%80%94%E2%80%94%E4%BB%A5ReentrantLock%E4%B8%BA%E4%BE%8B.assests/1576738298445.png)

4. 公平锁与非公平锁

   上面提到了`writerShouldBlock()`是公平锁与非公平锁的体现，这个函数的作用就是判断获取写锁是否需要阻塞，我们来看一下它的源码。

   `FairSync`：

   ```java
           final boolean writerShouldBlock() {
               // 返回队列中前面是否还有等待的，如果有就返回true，说明需要等待，果然公平！
               return hasQueuedPredecessors();
           }
   ```

   ```java
       public final boolean hasQueuedPredecessors() {
           Node t = tail; 
           Node h = head;
           Node s;
           // 这一大串的意思是：
           // 1.如果头节点和尾节点相等，说明要么队列是空的，要么前面只有一个头节点正持有锁，返回false，否则继续。（head节点是获取到锁的节点，但是任意时刻head节点可能占用着锁，也可能释放了锁（unlock()）,未被阻塞的head.next节点对应的线程在任意时刻都是有必要去尝试获取锁）
           // 2.令s = h.next，如果s==null，表示h!=t且h.next==null，什么情况下h!=t的同时h.next==null？？？这说明h后面肯定有其他节点，但是h.next又没有和这个节点连接成功，说明有其他线程正在加入阻塞队列，这种情况在上面的addWaiter()方法中解释过。返回true
           // 3.如果h!=t成立，head.next != null，则判断head.next是否是当前线程，如果是返回false，否则返回true。
           return h != t &&
               ((s = h.next) == null || s.thread != Thread.currentThread());
       }
   ```

   `NonfairSync`：

   ```java
           final boolean writerShouldBlock() {
               // 非公平锁任何时候都不等待，直接去抢，果然不公平！
               return false; 
           }
   ```

   除此之外，还有`readerShouldBlock() `方法，也来看一下：

   `FairSync`：

   ```java
           final boolean readerShouldBlock() {
               // 和writer一个意思
               return hasQueuedPredecessors();
           }
   ```

   `NonfairSync`：

   ```java
           final boolean readerShouldBlock() {
               // 这个apparentlyFirstQueuedIsExclusive()用来判断当前等待的第一个线程（链表中的第二个节点）是不是一个写线程
               // 如果是一个写线程就返回true，让这个读线程等待
               // 为什么要这样呢，既然是非公平锁直接去抢不就好了吗？我们知道，获取写锁比获取读锁的条件更为苛刻，如果有一个写线程在第一个位置等待，那么意味着这个写线程要和一个读线程竞争，那么大概率下这个写线程是会失败的，这样就有可能会造成该写线程长时间的等待。因此，如果读线程在看到等待的第一个线程是个写线程的话，就会“压制兽性，恢复人性”谦让以下。这种做法不是必须的，只是一种启发式的优化。
               return apparentlyFirstQueuedIsExclusive();
           }
   ```



### 3.1.2 释放锁

1. `unlock()`方法

   ```java
           public void unlock() {
               // 调用了AQS中的release方法
               sync.release(1);
           }
   ```

   ```java
       public final boolean release(int arg) {
           if (tryRelease(arg)) {
               Node h = head;
               if (h != null && h.waitStatus != 0)
                   unparkSuccessor(h);
               return true;
           }
           return false;
       }
   ```

   这个函数在介绍`ReentrantLock`时已经说所，就不赘述，关键在于`tryRelease()`方法，它是由子类实现的。

2. `tryRelease()`方法

   ```java
           protected final boolean tryRelease(int releases) {
               // 这个方法是五个模板方法之一，用来判断当前线程是否持有了排他锁
               // 如果不是，这个函数是释放排他锁的，抛出异常
               if (!isHeldExclusively())
                   throw new IllegalMonitorStateException();
               // 计算新的state值
               int nextc = getState() - releases;
               
               // free判断是否完全释放了排他锁
               boolean free = exclusiveCount(nextc) == 0;
               if (free)
                   // 将占有排他锁的线程设为null
                   setExclusiveOwnerThread(null);
               
               // 设置新的state
               setState(nextc);
               return free;
           }
   
           protected final boolean isHeldExclusively() {
               return getExclusiveOwnerThread() == Thread.currentThread();
           }
   ```



## 3.2 ReadLock

ReadLock实现的是共享锁模式。

### 3.2.1 获取锁

1. `lock()`方法

   ```java
           public void lock() {
               // 调用了AQS类中的acquireShared(1)方法
               sync.acquireShared(1);
           }
   ```

2. `acquireShared()`方法

   ```java
       public final void acquireShared(int arg) {
           // 首先tryAcquireShared(arg)尝试以自旋方式获取共享锁
           // 如果结果<0，表示失败，调用doAcquireShared(arg)在等待队列中等待获取共享锁
           if (tryAcquireShared(arg) < 0)
               doAcquireShared(arg);
       }
   ```

3. `tryAcquireShared()`方法

   这个方法需要派生类覆盖，是模板方法之一。

   ```java
           protected final int tryAcquireShared(int unused) {
               /*
                * 源码中这里有一段英文注释，解释这个函数的流程
                * 1. 如果有其他线程正持有写锁——失败
                * 2. 否则，此线程可能有资格获取锁，首先判断是否需要阻塞，如果不需要，尝试用CAS修改state。注意：这步没有检查重入请求，这是为了避免在不可重入的情况下去检查持有计数，重入检查被推迟到了fullTryAcquireShared逻辑。
                * 3. 如果第2步失败了，那么进入fullTryAcquireShared逻辑。
                */
               Thread current = Thread.currentThread();
               int c = getState();
               
               // 如果排他锁计数!=0，且持有排他锁的线程不是本线程——失败
               if (exclusiveCount(c) != 0 &&
                   getExclusiveOwnerThread() != current)
                   return -1;
               
               // 获取共享锁计数
               int r = sharedCount(c);
               
               // 首先判断是否需要阻塞，如果需要——fullTryAcquireShared
               // 然后检查共享锁计数是否已达上限，若是——fullTryAcquireShared
               // 使用CAS尝试获取共享锁，若失败——fullTryAcquireShared
               if (!readerShouldBlock() &&
                   r < MAX_COUNT &&
                   compareAndSetState(c, c + SHARED_UNIT)) {
                   // 这是获取共享锁成功的逻辑
                   
                   // 这一大串主要是为了更新HoldCount，HoldCount有什么用呢，这是为了实现jdk1.6中加入的getReadHoldCount()方法的，这个方法能获取当前线程重入共享锁的次数(state中记录的是多个线程的总重入次数)，加入了这个方法让代码复杂了不少，但是其原理还是很简单的：如果当前只有一个线程的话，还不需要动用ThreadLocal，直接往firstReaderHoldCount这个成员变量里存重入数，当有第二个线程来的时候，就要动用ThreadLocal变量readHolds了，每个线程拥有自己的副本，用来保存自己的重入数。
                   if (r == 0) {
                       firstReader = current;
                       firstReaderHoldCount = 1;
                   } else if (firstReader == current) {
                       firstReaderHoldCount++;
                   } else {
                       HoldCounter rh = cachedHoldCounter;
                       if (rh == null || rh.tid != getThreadId(current))
                           cachedHoldCounter = rh = readHolds.get();
                       else if (rh.count == 0)
                           readHolds.set(rh);
                       rh.count++;
                   }
                   
                   // 返回成功
                   return 1;
               }
               // 自旋获取锁
               return fullTryAcquireShared(current);
           }
   ```

4. `fullTryAcquireShared()`方法

   这个方法原理和`tryAcquireShared`类似，只不过加了自旋。

   ```java
           final int fullTryAcquireShared(Thread current) {
               HoldCounter rh = null;
               
               // 自旋
               for (;;) {
                   int c = getState();
                   if (exclusiveCount(c) != 0) {
                       if (getExclusiveOwnerThread() != current)
                           return -1;
                   } else if (readerShouldBlock()) {
                       // 注意，这一块非常重要，这就是在tryAcquireShared中所说的推迟检查重入请求。这一段是什么意思呢？意思就是说，如果当前线程在获取读锁的过程中需要被阻塞，并且当前线程不是重入请求的读锁，才能返回-1，让当前线程进入阻塞队列。这是因为如果当前线程是一个读锁重入的线程，又被加入了阻塞队列，那么如果阻塞队列中前面有一个写线程在等待，那么就会形成死锁！
                       
                       // 如果当前线程是第一个读线程那没事，因为第一个读线程的读锁重入次数不会为0
                       if (firstReader == current) {
                           // assert firstReaderHoldCount > 0;
                       } else {
                           // 否则获取当前线程的读锁重入次数
                           if (rh == null) {
                               rh = cachedHoldCounter;
                               if (rh == null || rh.tid != getThreadId(current)) {
                                   rh = readHolds.get();
                                   if (rh.count == 0)
                                       readHolds.remove();
                               }
                           }
                           // 如果读锁重入次数为0，才能返回-1，进入阻塞队列。
                           if (rh.count == 0)
                               return -1;
                       }
                   }
                   if (sharedCount(c) == MAX_COUNT)
                       throw new Error("Maximum lock count exceeded");
                   if (compareAndSetState(c, c + SHARED_UNIT)) {
                       if (sharedCount(c) == 0) {
                           firstReader = current;
                           firstReaderHoldCount = 1;
                       } else if (firstReader == current) {
                           firstReaderHoldCount++;
                       } else {
                           if (rh == null)
                               rh = cachedHoldCounter;
                           if (rh == null || rh.tid != getThreadId(current))
                               rh = readHolds.get();
                           else if (rh.count == 0)
                               readHolds.set(rh);
                           rh.count++;
                           cachedHoldCounter = rh; // cache for release
                       }
                       return 1;
                   }
               }
           }
   ```

5. `doAcquireShared()`方法

   这个方法是`AQS`中的方法，在等待队列中等待获取共享锁。

   ```java
       private void doAcquireShared(int arg) {
           final Node node = addWaiter(Node.SHARED);
           boolean failed = true;
           try {
               boolean interrupted = false;
               for (;;) {
                   final Node p = node.predecessor();
                   if (p == head) {
                       int r = tryAcquireShared(arg);
                       if (r >= 0) {
                           setHeadAndPropagate(node, r);
                           p.next = null; // help GC
                           if (interrupted)
                               selfInterrupt();
                           failed = false;
                           return;
                       }
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

   





