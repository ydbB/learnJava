<center>并发包中ConcurrentLinkedQueue 和 LinkedBlockingQueue 有什么区别</center>
问题20：并发包中ConcurrentLinkedQueue 和 LinkedBlockingQueue 有什么区别？

### 典型回答
习惯认为，并发包下的所有容器都叫并发容器，但是实际上只有像 ConcurrentLinkedQueue 这样的”Concurrent*“容器才代表并发。
他们之间的区别：
* Concurrent 基于 lock-free，常见于多线程访问场所，一般可以提供较高的吞吐量；
* 而 LinkedBlockingQueue 内部是基于锁实现的，并提供了 BlockingQueue 的等待方法；

java.util.concurrent 包中提供的容器按照命名，可以分为 Concurrent，CopyOnWrite 和 Blocking 三类：
* Concurrent 没有 CopyOnWrite 容器之类相对较重的修改开销；
* 但是，因此 Concurrent 也就只能提供弱一致性，例如在遍历的时候，容器修改了，但是遍历还能继续；
* 与弱一致性相对的就是常见的”fail-fast“机制，如果在遍历中容器被修改了，则立即抛出 ConcurrentModifcationException ，并停止遍历。
* 弱一致性的另外一个体现就是，size 等操作的准确性是有限的；
* 同时读取的性能也有一定不准确性。

### 考点分析
这个问题，考察了并发包下不同容器的设计目的和实现。

队列是非常重要的数据结构，日程开发中很多线程数据传递都需要依赖它，同时 Executor 框架提供发各种线程池，离不开它。面试管可以考察：
* 哪些队列是有界的？哪些是无解的？
* 针对特定的场景需求，如何选择合适的队列实现？
* 从源码的角度，常见的线程安全队列是如何实现的，并进行了哪些改进以提高性能表现？


### 知识扩展

Queue是否有界：
* ArrayBlockingQueue 是最典型的的有界队列，其内部以 final 的数组保存数据，数组的大小就决定了队列的边界，所以我们在创建 ArrayBlockingQueue 时，都要指定容量；
* LinkedBlockingQueue，容易被误解为无边界，但其实其行为和内部代码都是基于有界的逻辑实现的，只不过如果我们没有在创建队列时就指定容量，那么其容量限制就自动被设置为 Integer.MAX_VALUE，成为了无界队列。
* SynchronousQueue，这是一个非常奇葩的队列实现，每个删除操作都要等待插入操作，反之每个插入操作也都要等待删除动作。那么这个队列的容量是多少呢？是 1 吗？其实不是的，其内部容量是 0。
* PriorityBlockingQueue 是无边界的优先队列，虽然严格意义上来讲，其大小总归是要受系统资源影响。
* DelayedQueue 和 LinkedTransferQueue 同样是无边界的队列。对于无边界的队列，有一个自然的结果，就是 put 操作永远也不会发生其他 BlockingQueue 的那种等待情况。

日常开发中，如何进行队列选择：
* 考虑应用场景中对队列边界的要求。ArrayBlockingQueue 是有明确的容量限制的，而 LinkedBlockingQueue 则取决于我们是否在创建时指定，SynchronousQueue 则干脆不能缓存任何元素。
* 从空间利用角度，数组结构的 ArrayBlockingQueue 要比 LinkedBlockingQueue 紧凑，因为其不需要创建所谓节点，但是其初始分配阶段就需要一段连续的空间，所以初始内存需求更大。
* 通用场景中，LinkedBlockingQueue 的吞吐量一般优于 ArrayBlockingQueue，因为它实现了更加细粒度的锁操作。
* ArrayBlockingQueue 实现比较简单，性能更好预测，属于表现稳定的“选手”。
* 如果我们需要实现的是两个线程之间接力性（handoff）的场景，按照专栏上一讲的例子，你可能会选择 CountDownLatch，但是SynchronousQueue也是完美符合这种场景的，而且线程间协调和数据传输统一起来，代码更加规范。
* 可能令人意外的是，很多时候 SynchronousQueue 的性能表现，往往大大超过其他实现，尤其是在队列元素较小的场景。