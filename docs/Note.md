# 笔记
- 线程安全的代码：管理state的访问，一般是可变、共享的状态。
  - 可变：变量的值可以更改
  - 共享：变量可以被多个线程访问
- 保证线程安全的方法
  - 不要跨线程共享变量
  - 使用不可变的state变量
  - 访问state变量时使用同步
---
- 类线程安全的定义：被多个线程访问时，类的行为依然正确。
- 线程安全的类已经封装了同步，客户端不需要自己提供。
- 无状态的对象永远线程安全。
- 为保证state一致性，需要在原子操作中更新相关联的state变量。
- java 强制原子性的内部锁机制：synchronized 块。
- 每个共享的state变量都需要唯一一个确定的锁来保护，维护者必须知道这个锁。
- 对于每一个涉及变量的不变约束，需要同一个锁保护其所有的变量。参阅 SynchronizedFactorizer
---
- 锁代表着：同步/互斥(原子性) + 内存可见性。保证一个线程对数值的写入，其他线程都可见。
- volatile仅仅代表：内存可见性。
- 线程封闭：如果数据只在单线程中被访问，就不要任何的同步。线程封闭保证了原子性。
- 如果确保单一线程写入一个共享volatile变量i，那么i++也就是线程安全的。因为单一线程保证原子性，volatile本身提供可见性。
- ThreadLocal 确保线程封闭：get总是返回当前线程通过set设置的最新值。
- 不可变对象的性质
  - 它的state在创建后无法修改
  - 所有filed都是final
  - 它被正确地创建（创建时this没有逃逸出去）
- 最佳实践：将field设置为private，除非你想暴露它；将field设置为final，除非它是可变的。
- 如果final指向可变对象，访问这些对象的state时依然需要同步。
- 安全发布的模式：对象的引用和对象的state必须同时对其他线程可见
  - 静态初始化器初始化对象的引用
  - 将该引用保存到volatile或者AtomicReference的field
  - 将该引用保存到正确对象的final域中
  - 或者，将该引用保存到由锁保护的域中
- 静态发布：`public static Holder holder = new Holder(32);` JVM在类的初始化阶段执行，由JVM内在的同步保证安全发布。
- 共享对象的有效策略
  - 线程限制：只能被占有它的线程修改
  - 共享只读：任何线程都不能修改它
  - 共享线程安全：一个线程安全的对象在内部进行同步
  - 被守护：一个被守护的对象只能通过特定的锁来访问。
---
- 设计线程安全的类
  - 确定对象state变量有哪些
  - 确定限制state变量的不变约束
  - 指定管理并发访问对象state的策略
- 对象的state取决于它的field。Counter 只有一个value，那么state就只有一个。
- 理解对象的不变约束和后验条件，才能保证线程安全。
- state依赖的操作：无法从空队列取元素，无法向满队列放元素。
- 将数据封装于对象内部，把对数据的访问限制于对象的方法上。
- ArrayList和HashMap不是线程安全，通过包装器工厂方法Collections.synchronizedList等可以
保证线程安全。原理：利用修饰器模式将相关API方法实现为同步，请求转发到底层容器。
- 如果一个类有多个独立的线程安全的state变量组成，类也不包含state转换，
那么可以将线程安全委托给这些变量。
- 客户端加锁必须保证使用的锁和对象X保护自身state的锁一样。
- 同步策略的文档化：为类的用户编写线程安全性担保的文档，为类的维护者编写类的同步策略的文档。
---
- 同步容器的迭代器和ConcurrentModificationException：及时失败策略，察觉到被别人修改了，就果断报错。
  - 原理：使用modCount记录修改次数，迭代期间，如果发现该计数器数值变了，代表其他线程做出影响长度的修改，抛出CME。
- 迭代期间，对容器加锁的替代方法是复制容器。CopyOnWrite ?
- 同步容器对容器所有state进行串行访问，同步粒度过大，影响了并发性，也就是削弱了性能。
- JDK 5.0+引入并发容器，例如ConcurrentHashMap和CopyOnWriteArrayList，ConcurrentLinkedQueue
- JDK 6.0+引入ConcurrentSkipListMap和ConcurrentSkipListSet，作为对SortMap和SortSet的并发替代。
- JDK 7中，ConcurrentHashMap使用分段锁机制，每一段一个锁，支持有限数量的写线程并发修改。
  - 它的迭代器是弱一致性的，允许并发修改，可以（但不保证）能看到迭代器创建之后，容器的修改。
  - size的返回值是不精确的。
  - 具有原子操作：putIfAbsent，removeIfEqual，replaceIfEqual
  - 缺点：无法独占加锁。
- CopyOnWriteList：写时复制的迭代器返回的元素和迭代器创建时严格一致，不会考虑后续的修改。
- BlockingQueue 提供可阻塞的put和take方法，与定时的offer、poll是等价的。
- BlockingQueue 的实现有LinkedBlockingQueue、ArrayBlockingQueue和SynchronousQueue。
- 对象池拓展了连续的线程限制，将“对象”借给一个请求线程。
- 双端队列：JDK 6.0+新增Deque和Blocking1Queue。Deque是双端队列，实现为ArrayDeque和LinkedBlockingDeque。
- 工作窃取：一个消费者完成自身双端队列的任务后，可以去其他消费者的双端队列尾部偷任务。
- Thread 提供了中断机制。一个线程不能强迫其他线程停止正在做的事情，或者去做其他事情。
A中断B，仅仅代表A要求B在达成某个方便停止的关键点时，停止正在做的事情。
- 一个方法可以抛出InterruptedException时，代表这是一个可阻塞的方法。
- 响应中断的策略
  - 向上级传递InterruptedException
  - 捕获InterruptedException，在当前线程调用interrupt()恢复中断。恢复中断是为了不吞掉这个中断，避免掩盖中断。
  - 注意，不应该捕获它，然后什么都不做。
- synchronizer是一个对象，根据本身state调节线程控制流。
  - 阻塞队列可以扮演sync角色；其他synchronizer包括Semaphore（信号量）、barrier（关卡），latch（闭锁）。
  - sync封装state，决定着线程在某个时间点汇聚时，是通过或者等待。
- 闭锁（latch），可以延迟线程进度到终止状态。开始阀门能同时释放所有工作者线程，结束阀门可以等待最后一个线程执行完毕。
- FutureTask