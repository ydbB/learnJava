<center>Java有哪几种文件拷贝方式？哪一种最高效？</center>
问题：Java有哪几种文件拷贝方式？哪一种最高效？
### 典型回答
Java今天有多种文件拷贝方式，如：
1.利用java.io库，直接为源文件构建一个FileInputStream读取，为目标构建一个FileOutStream,完成写入工作。
```java

public static void copyFileByStream(File source, File dest) throws
        IOException {
    try (InputStream is = new FileInputStream(source);
         OutputStream os = new FileOutputStream(dest);){
        byte[] buffer = new byte[1024];
        int length;
        while ((length = is.read(buffer)) > 0) {
            os.write(buffer, 0, length);
        }
    }
 }
```  
2.利用java.noi类库，提供的transferTo和transFrom.
```java

public static void copyFileByChannel(File source, File dest) throws
        IOException {
    try (FileChannel sourceChannel = new FileInputStream(source)
            .getChannel();
         FileChannel targetChannel = new FileOutputStream(dest).getChannel
                 ();){
        for (long count = sourceChannel.size() ;count>0 ;) {
            long transferred = sourceChannel.transferTo(
                    sourceChannel.position(), count, targetChannel);            sourceChannel.position(sourceChannel.position() + transferred);
            count -= transferred;
        }
    }
 }
```  
### 考点分析
今天这个问题，从面试的角度来看，确实是一个面试考察的点，针对我上面的典型回答，面试官还可能会从实践角度，或者 IO 底层实现机制等方面进一步提问。这一讲的内容从面试题出发，主要还是为了让你进一步加深对 Java IO 类库设计和实现的了解。  
从技术的角度展开，需要注意：
* 不同copy方式，底层的实现机制有什么不同？
* 为什么零拷贝有性能优势？
* Buffer的使用和分类
* Direct Buffer 对垃圾收集的影响和实践选择
### 知识扩展
1.拷贝实现机制分析
用户态空间：给普通应用和服务；
内核态空间：具有较高的权限。
* 使用输入输出流进行读写时， 其实进行了多次上下文切换，比如读文件时，内核态从磁盘中读取数据到内核缓存，用户态在将数据懂内核缓存读取到用户缓存，写的过程类似。
* 基于NIO的transf技术在Linux和UNIX上会使用零拷贝技术，数据拷贝不需要要用户态参与，节省了上下文切换和内存拷贝，从而提高了数据拷贝的性能。
2.Java IO/NIO的源码结构
java标准类库也提供文件拷贝的方法，java.nio.file.Files.copy.
如何提高拷贝性能的一些宽泛原则：
* 在程序中，使用缓存等机制，合理减少 IO 次数（在网络通信中，如 TCP 传输，window 大小也可以看作是类似思路）；
* 使用 transferTo 等机制，减少上下文切换和额外 IO 操作。
* 尽量减少不必要的转换过程，比如编解码；对象序列化和反序列化，比如操作文本文件或者网络通信，如果不是过程中需要使用文本信息，可以考虑不要将二进制信息转换成字符串，直接传输二进制信息。
3.掌握NIO Buffer
Buffer 是 NIO 操作数据的基本工具，Java 为每种原始数据类型都提供了相应的 Buffer 实现（布尔除外），所以掌握和使用 Buffer 是十分必要的，尤其是涉及 Direct Buffer 等使用，因为其在垃圾收集等方面的特殊性，更要重点掌握。  
Buffer的几个基本特性：
* capacity，它反映这个 Buffer 到底有多大，也就是数组的长度。
* position，要操作的数据起始位置。
* limit，相当于操作的限额。在读取或者写入时，limit 的意义很明显是不一样的。比如，读取操作时，很可能将 limit 设置到所容纳数据的上限；而在写入时，则会设置容量或容量以下的可写限度。
* mark，记录上一次 postion 的位置，默认是 0，算是一个便利性的考虑，往往不是必须的。
Buffer的基本 操作：
* 我们创建了一个 ByteBuffer，准备放入数据，capacity 当然就是缓冲区大小，而 position 就是 0，limit 默认就是 capacity 的大小。
* 当我们写入几个字节的数据时，position 就会跟着水涨船高，但是它不可能超过 limit 的大小；
* 如果我们想把前面写入的数据读出来，需要调用 flip 方法，将 position 设置为 0，limit 设置为以前的 position 那里。
* 如果还想从头再读一遍，可以调用 rewind，让 limit 不变，position 再次设置为 0。
4. Direct Buffer 和垃圾收集
   重点介绍两种特别的 Buffer：
   * Direct Buffer：如果我们看 Buffer 的方法定义，你会发现它定义了 isDirect() 方法，返回当前 Buffer 是否是 Direct 类型。这是因为 Java 提供了堆内和堆外（Direct）Buffer，我们可以以它的 allocate 或者 allocateDirect 方法直接创建。
   * MappedByteBuffer：它将文件按照指定大小直接映射为内存区域，当程序访问这个内存区域时将直接操作这块儿文件数据，省去了将数据从内核空间向用户空间传输的损耗。我们可以使用FileChannel.map创建 MappedByteBuffer，它本质上也是种 Direct Buffer。
  在实际使用中，Java 会尽量对 Direct Buffer 仅做本地 IO 操作，对于很多大数据量的 IO 密集操作，可能会带来非常大的性能优势：
  * Direct Buffer 生命周期内内存地址都不会再发生更改，进而内核可以安全地对其进行访问，很多 IO 操作会很高效。
  * 减少了堆内对象存储的可能额外维护工作，所以访问效率可能有所提高。
  Direct Buffer 创建和销毁过程中，都会比一般的堆内 Buffer 增加部分开销，所以通常都建议用于长期使用、数据较大的场景。
  Direct Buffer，我们需要清楚它对内存和 JVM 参数的影响。首先，因为它不在堆上，所以 Xmx 之类参数，其实并不能影响 Direct Buffer 等堆外成员所使用的内存额度，我们可以使用下面参数设置大小：
  ```
  
-XX:MaxDirectMemorySize=512M

```  
对于 Direct Buffer 的回收，几个建议：
* 在应用程序中，显式地调用 System.gc() 来强制触发;
* 另外一种思路是，在大量使用 Direct Buffer 的部分框架中，框架会自己在程序中调用释放方法，Netty 就是这么做的;
* 重复使用 Direct Buffer.
5. 跟踪和诊断 Direct Buffer 内存占用？
幸好，在 JDK 8 之后的版本，我们可以方便地使用 Native Memory Tracking（NMT）特性来进行诊断，你可以在程序启动时加上下面参数：
```

-XX:NativeMemoryTracking={summary|detail}

```  
激活 NMT 通常都会导致 JVM 出现 5%~10% 的性能下降.