<center>synchronized 和 ReentrantLock有什么区别？</center>
软件并发已成为当代软件开发的基础能力，而Java精心设计的高效并发机制，正是构建大规模应用的基础之一。
问题15：synchronized 和 ReentrantLock有什么区别？有人说synchronized慢，这话靠谱吗？
### 典型回答
1.synchronized是Java内建的同步机制，它提供了互斥的语义和可见性，当一个线程获取占有锁时，其他想要这个锁的线程只能阻塞等待；
2.ReentrantLock在Java 5中引入的锁实现，语义与synchronized类似，通过代码直接调用lock（）来实现。ReentrantLock提供了很多实用的方法，可以实现很多synchronized无法实现的细节，可以控制 fairness，也就是公平性，或者利用定义条件等。但是，编码中也需要注意，必须要明确调用 unlock() 方法释放，不然就会一直持有该锁。
synchronized 和 ReentrantLock 的性能不能一概而论，早期版本 synchronized 在很多场景下性能相差较大，在后续版本进行了较多改进，在低竞争场景中表现可能优于 ReentrantLock。
### 考点分析
锁作为并发的基础工具之一，你至少需要掌握：
* 理解什么是线程安全。
* synchronized、ReentrantLock 等机制的基本使用与案例。
* 掌握 synchronized、ReentrantLock 底层实现；理解锁膨胀、降级；理解偏斜锁、自旋锁、轻量级锁、重量级锁等概念。
* 掌握并发包中 java.util.concurrent.lock 各种不同实现和案例分析。
### 知识扩展
线程安全是一个多线程环境下正确性的概念，也就是保证多线程环境下共享的、可修改的状态的正确性，这里的状态反映在程序中其实可以看作是数据。  
线程安全需要保证几个基本特性：
* 原子性，简单说就是相关操作不会中途被其他线程干扰，一般通过同步机制实现。
* 可见性，是一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到主内存上，volatile 就是负责保证可见性的。
* 有序性，是保证线程内串行语义，避免指令重排等。
