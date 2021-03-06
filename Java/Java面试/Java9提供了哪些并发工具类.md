<center>Java 9 并发包 提供了哪些并发工具类</center>
问题19： Java 9 并发包 提供了哪些并发工具类
### 典型回答
通常所说的并发包就是指java.util.concurrent 及其子包，集中了Java各种基础工具，主要体现在：
* 提供了比 synchronized 更加高级的各种同步结构，包括 CountDownLatch、CyclicBarrier、Semaphore 等，可以实现更加丰富的多线程操作；
* 各种线程安全的容器，比如最常见的 ConcurrentHashMap、有序的 ConcurrentSkipListMap，或者通过类似快照机制，实现线程安全的动态数组 CopyOnWriteArrayList 等。
* 各种并发队列实现，如各种 BlockingQueue 实现，比较典型的 ArrayBlockingQueue、 SynchronousQueue 或针对特定场景的 PriorityBlockingQueue 等。
* 强大的 Executor 框架，可以创建各种不同类型的线程池，调度任务运行等，绝大部分情况下，不再需要自己从头实现线程池和任务调度器。
### 考点分析
这个题目主要考察对于并发包的了解程度，是否在实际工程中使用过。我们使用并发包的目的主要是：
* 利用多线程提高程序的扩展能力，以达到业务对吞吐量的要求。
* 协调线程间调度、交互，以完成业务逻辑。
* 线程间传递数据和状态，这同样是实现业务逻辑的需要。
这块知识主要需要掌握：
* 从总体上，把握住几个主要组成部分（前面回答中已经简要介绍）。
* 理解具体设计、实现和能力。
* 再深入掌握一些比较典型工具类的适用场景、用法甚至是原理，并熟练写出典型的代码用例。
### 知识扩展
