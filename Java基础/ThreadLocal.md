## ThreadLocal介绍

java.lang.ThreadLocal无法解决共享对象的更新问题，ThreadLocal对象建议使用static修饰。这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。

同一个 ThreadLocal 所包含的对象，在不同的 Thread 中有不同的副本

#### 常用方法

- set(T value)：设置线程本地变量的内容。
- get()：获取线程本地变量的内容。
- remove()：移除线程本地变量。注意在线程池的线程复用场景中在线程执行完毕时一定要调用remove，避免在线程被重新放入线程池中时被本地变量的旧状态仍然被保存。

#### 实现原理

![](https://www.zdk.world/wp-content/uploads/2022/08/ThreadLocal.drawio.png)

- java.lang.Thread类中有成员变量threadLocals(线程独有,每个线程都有独立的threadLocals,实现了线程间的数据隔离),类型为ThreadLocal的内部类ThreadLocalMap
- ThreadLoaclMap中有成员变量table,类型为其内部类Entry数组,初始大小为16,每次扩容翻倍(阈值为当前size的2/3);代码中定义的各种ThreadLocal对象的值均存储在该table中
- Entry中保存实际数据,类型为k-v结构,其中k为ThreadLocal类型(弱引用);

#### 内存泄漏问题

**存在内存泄漏的地方:**

- ThreadLocal:ThreadLocal对象引用都被回收之后,由于Entry中仍存在对该ThreadLocal对象的引用而不会回收该对象(这点由于Entry中对ThreadLocal的**引用为弱引用**而避免;可Entry中的value会出现未被回收的情况)
- Entry中的value:`ThreadLocalMap的生命周期与Thread一致，如果不手动清除掉Entry对象的话就可能会造成内存泄露问题。`因此，在每次在使用完之后需要手动的remove掉Entry对象。

**如何避免内存泄漏:**

- 每次ThreadLocal使用完毕后,手动调用remove方法移除掉Entry对象;
- 使用完ThreadLocal后线程也随之结束(会释放掉threadLocalMap),如果是线程池则可能不会释放;

Entry中的key使用**弱引用**的原因就是避免ThreadLocal使用之后未手动remove掉,而导致(ThreadLocal对象)内存泄漏问题

> - ThreadLocalMap是ThreadLocal的内部类，它是一个定制化的哈希表，用来存储线程本地变量。它的键是ThreadLocal对象，它的值是线程本地变量的值。ThreadLocalMap的Entry类继承了WeakReference类，因此它对键的引用是弱引用，而对值的引用是强引用。
> - ThreadLocalMap的键为弱引用的好处是，当ThreadLocal对象没有被其他地方强引用时，它就可以被垃圾回收器回收，这样可以避免内存泄漏的风险。如果键为强引用，那么即使ThreadLocal对象不再使用，它也会一直存在于ThreadLocalMap中，占用内存空间。
> - ThreadLocalMap的值为强引用的原因是，如果值为弱引用，那么即使ThreadLocal对象还在使用，它也可能被垃圾回收器回收，这样就会导致数据丢失或错误。因为对我们来说，值才是我们想要保存的数据，ThreadLocal只是用来关联值的，如果值都没了，还要ThreadLocal干嘛呢？
> - 但是，即使键为弱引用，也不能完全避免内存泄漏的问题。因为当键被回收后，值还在ThreadLocalMap中存在，而且无法访问或删除。所以，在使用完ThreadLocal后，最好调用remove方法来清理掉对应的Entry对象。另外，在ThreadLocalMap中进行set和get操作时，也会进行一些清理工作，去除那些键为null的Entry对象。

#### InheritableThreadLocal

可继承的ThreadLocal:当线程中的ThreadLocal对象值需要共享给子线程时使用InheritableThreadLocal来实现

**实现原理:**

- Thread中存在成员变量:ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
- InheritableThreadLocal继承了ThreadLocal,重写了createMap使用了线程中的inheritableThreadLocals 变量
- Thread的初始化方法init完成inheritableThreadLocals 变量的初始化(将父线程的threadLocalMap内容拷贝给自己一份)
- **如果Thread的变量inheritableThreadLocals 对象实例化之后,父线程再次修改其中的值,子线程读取的仍为旧值**