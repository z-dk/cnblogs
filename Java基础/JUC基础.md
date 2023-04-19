# JUC

Java并发包:java.util.concurrent

**volatile**:内存可见性,保证了线程安全三要素中的有序性,可见性,不保证原子性([代码示例](https://github.com/z-dk/helloworld/blob/master/hello-web/src/test/java/juc/VolatileTest.java))

**内存屏障可以解决可见性和有序性的问题**。内存屏障可以强制刷新缓存，使得其他线程可以看到最新的值，从而保证可见性。

- 可见性:内存屏障,当CPU写数据时如果发现变量在其他CPU中存在副本,那么会发出信号通知其他CPU将该副本对应的缓存置为无效,其他CPU发现缓存无效时会从主存中重新获取
- 有序性:内存屏障也可以限制指令重排的范围，使得在屏障之前的操作不会被重排到屏障之后，从而保证有序性。

**CAS**:(Compare-And-Swap)保证原子性,存在两个问题:

- ABA问题:引入版本号解决
- 自旋:

**锁分段机制**:提供了以下几种实现

- ConcurrentHashMap 通常优于同步的 HashMap(1.8 以后底层锁分段又换成了CAS)
- ConcurrentSkipListMap 通常优于同步的 TreeMap
- CopyOnWriteArrayList 优于同步的 ArrayList(当期望的读数和遍历远远大于列表的更新数时)

**CountDownLatch 闭锁**:在完成某些运算时，只有其他所有线程的运算全部完成，当前运算才继续执行

- 确保某个计算在其需要的所有资源都被初始化之后才继续执行;
- 确保某个服务在其依赖的所有其他服务都已经启动之后才启动;
- 等待直到某个操作所有参与者都准备就绪再继续执行。

**CyclicBarrier栅栏**:各个线程通过栅栏实现在某一状态后(cyclicBarrier.await等待,直到所有线程都await后,线程数量由barrier初始化时指定)同时继续执行;

```java
public static void main(String[] args) {
        // 所有线程在cyclicBarrier的wait后停下,等待最后一个wait,之后一起继续执行
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> System.out.println("barrier"));
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        executorService.submit(() -> {
            System.out.println("=== 1 ===start");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("=== 1 ===end");
        });
        executorService.submit(() -> {
            System.out.println("=== 2 ===start");
            try {
                Thread.sleep(3000);
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("=== 2 ===end");
        });
        executorService.submit(() -> {
            System.out.println("=== 3 ===start");
            try {
                Thread.sleep(5000);
                cyclicBarrier.await();
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            System.out.println("=== 3 ===end");
        });
    }
```

**Semaphore信号量**:多线程申请一定数量的资源,资源存在申请成功并扣除相应数量的资源,资源不够则阻塞直到资源充足,release释放一定数量的资源(可以不等于申请的资源数量)

```java
public static void main(String[] args) {
        // 线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> LOGGER.info("barrier"));
        // 信号量
        Semaphore semaphore = new Semaphore(3);
        
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        executorService.submit(() -> {
            LOGGER.info("=== 1 ===start");
            try {
                cyclicBarrier.await();
                LOGGER.info("=== 1 ===weak up");
                Thread.sleep(3000);
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            LOGGER.info("=== 1 ===end");
        });
        executorService.submit(() -> {
            LOGGER.info("=== 2 ===start");
            try {
                Thread.sleep(3000);
                cyclicBarrier.await();
                LOGGER.info("=== 2 ===weak up");
                Thread.sleep(1000);
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            LOGGER.info("=== 2 ===end");
        });
        executorService.submit(() -> {
            LOGGER.info("=== 3 ===start");
            try {
                Thread.sleep(5000);
                cyclicBarrier.await();
                LOGGER.info("=== 3 ===weak up");
                Thread.sleep(2000);
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
            LOGGER.info("=== 3 ===end");
        });
        executorService.shutdown();
    }
```

**Callable 接口**:类似于Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是 Runnable 不会返回结果，并且无法抛出受检异常。

**Thread的start与run**:单独调用run()方法，是同步执行；通过start()调用run()，是异步执行。

**线程池的submit与execute**:execute只接受runnable且无返回值,submit接受runnable和callable,有返回值,但如果不get返回值,会吞掉内部的异常.

**wait/notify/notifyall**:使用wait应在循环中(避免虚假唤醒问题:多个wait被同时唤醒)

**Condition**:控制线程通信,单个Lock可与多个Condition对象关联,对应:await、 signal 和 signalAll

**ReadWriteLock 读写锁**:ReadWriteLock接口(`ReentrantReadWriteLock`)维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

**StampedLock**:该锁提供了三种模式的读写控制，当调用获取锁的系列函数的时候，会返回一个long 型的变量，该变量被称为戳记（stamp),这个戳记代表了锁的状态。try系列获取锁的函数，当获取锁失败后会返回为0的stamp值。当调用释放锁和转换锁的方法时候需要传入获取锁时候返回的stamp值。

StampedLock 提供的读写锁与 ReentrantReadWriteLock 类似，只是前者的都是不可重入锁。但是前者通过提供乐观读锁在多线程多读的情况下提供更好的性能，这是因为获取乐观读锁时候不需要进行 CAS 操作设置锁的状态，而只是简单的测试状态。

| 方法                                              | 描述               | 备注                                                         |
| ------------------------------------------------- | ------------------ | ------------------------------------------------------------ |
| writeLock/readLock                                | 阻塞获取写锁       | 阻塞直至获取到锁                                             |
| tryWriteLock/tryReadLock                          | 非阻塞获取写锁     | 可指定超时时间,获取不到返回0,即失败                          |
| writeLockInterruptibly<br />readLockInterruptibly | 阻塞获取写锁       | 可响应中断                                                   |
| tryOptimisticRead                                 | 乐观读锁           | 在操作数据前并没有通过 CAS 设置锁的状态，仅仅是通过位运算测试；如果当前没有线程持有写锁，则简单的返回一个非 0 的 stamp 版本信息,后续需使用validate验证锁释放可用 |
| tryConvertToWriteLock                             | 将当前锁转换为写锁 | 当前如果为写锁直接返回当前票据,否则返回写锁票据<br />tryConvertToOptimisticRead<br />tryConvertToReadLock |

**synchronized**:某一个时刻内，只要一个线程去调用其中的一个synchronized方法了，其它的线程都只能等待.

- 某一个时刻内，只能有唯一一个线程去访问这些synchronized方法
- 锁的是当前对象this，被锁定后，其它的线程都不能进入到当前对象的其它的synchronized方法
- 所有的非静态同步方法用的都是同一把锁——**实例对象本身**
- 所有的静态同步方法用的也是同一把锁——**类对象本身**

**ForkJoinPool 分支/合并框架**:将一个大任务，进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行 join 汇总。

- 当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。
- 相对于一般的线程池实现， fork/join 框架的优势体现在对其中包含的任务的处理方式上。在一般的线程池中， 如果一个线程正在执行的任务由于某些原因无法继续运行， 那么该线程会处于等待状态。而在fork/join框架实现中，如果某个子问题由于等待另外一个子问题的完成而无法继续运行。那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程的等待时间， 提高了性能。
