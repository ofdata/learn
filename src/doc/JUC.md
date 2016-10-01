## 1. 原子类java.util.concurrent.atomic.*原理分析

在并发编程下，原子操作类的应用可以说是无处不在的。为解决线程安全的读写提供了很大的便利。

原子类保证原子的两个关键的点就是：可见性和写数据一致性。

### 对修改可见
使用volatile来保证读取到最新的数据。
**volatile语义：** 用简单的文字来讲，volatile保证了Java共享变量在多线程环境下对读可见的特性。因为它不是Java语言级别的锁，所以不会造成上下文切换，使用恰当的情况下比锁有更好的性能。
底层原理：
volatile 是在CPU层面上实现保证了数据的可见性。在写入数据的时候，处理CPU会从系统内存中把数据读到CPU缓存里进行修改，同时回写到系统内存，在这个过程中，其他CPU就不能访问共享内存，直到处理
CPU完成。写回内存后会导致其他CPU缓存失效，所以所有的CPU看到的都会是新的值。

### 写数据一致性
底层通过使用**乐观锁 + CAS**的方式进行原子更新。
**CAS：** 在原子类这个包下的所有的CAS操作都是使用```sun.misc.Unsafe```的这个类。在Hotspot虚拟机下这个类内部用JNI方式调用底层实现的。通过CPU指令```cmpxchg```保证了原子性。


## 2. 线程池原理分析

方法定义：
```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

核心流程在execute方法，执行步骤为：
1. 如果核心线程数没有满，则直接新建一个Worker（内部包含一个新的线程），执行执行任务，注意，不需要入队列。
2. 如果核心线程数满了或者没有Worker添加没有成功，则判断先当前是否为非运行状态，如果是并且将当前任务从workQueue中移除成功，则触发拒绝策略。如果是正常运行状态，并且空闲线程数为0，则尝试添加Worker。
3. 如果线程池没有在运行或者最终没有成功进入队列，则尝试创建新的Worker并且将任务传入。如果添加失败，则触发拒绝策略。