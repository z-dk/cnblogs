## AQS队列同步器

`java.util.concurrent.locks.AbstractQueuedSynchronizer`是一个同步器+阻塞锁的基本架构,用于控制加锁和释放锁,并在内部维护一个FIFO的线程等待队列,juc包下的锁,屏障等同步器多数是基于它实现的.

![](https://img2023.cnblogs.com/blog/1457262/202305/1457262-20230503094846237-318412321.png)

AQS每当有新的线程请求资源时,该线程就会进入一个等待队列(Waiter Queue),只有持有锁的线程释放资源后,该线程才能持有资源;

等待队列的实现方式为双向链表,线程会被包装在链表节点Node中

### Sync Queue同步队列

同步队列是用来管理获取锁的线程

- 同步队列，用来存放获取锁失败的线程，是一个双向链表，每个Node节点都有prev和next指针，指向前驱和后继节点。
- 同步队列有两个属性head和tail，分别指向队列的头节点和尾节点。
- 当一个线程获取锁失败后，会调用addWaiter方法，将自己包装成一个Node节点，并加入到同步队列的尾部，然后调用acquireQueued方法，进入自旋循环，等待被唤醒或者重新尝试获取锁。
- 当一个线程释放锁后，会调用unparkSuccessor方法，唤醒同步队列中的头节点的后继节点，让其重新竞争锁。

### Condition Queue条件队列

条件队列是管理等待条件的线程

- 条件队列，用来存放调用await方法的线程，是一个单向链表，每个Node节点都有nextWaiter指针，指向下一个节点。
- 条件队列有两个属性firstWaiter和lastWaiter，分别指向队列的第一个节点和最后一个节点。
- 当一个线程已经获取了锁，但是需要等待某个条件满足时，会调用await方法，将自己包装成一个Node节点，并加入到条件队列的尾部，然后调用fullyRelease方法，释放已经获取的锁，并进入等待状态。
- 当某个线程满足了条件，并调用了signal或者signalAll方法后，会调用doSignal或者doSignalAll方法，将条件队列中的头节点或者所有节点转移到同步队列中，让其重新竞争锁。