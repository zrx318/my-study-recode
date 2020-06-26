##知识扫盲：什么是零拷贝（Zero-copy）
- 零拷贝，就是在操作数据时，不需要将数据buffer从一个内存区域拷贝到另一个内存区域。因为少了一次内存的拷贝，因此CPU的效率就得到了提升。
- 在OS层面上的Zero-copy通常指避免在用户态和内核态之间来回拷贝数据。例如Linux提供的mmap系统调用，它可以将一段用户空间内存映射到内核空间，当映射成功后，用户对这段内存区域的修改可以直接反映到内核空间；同样，内核空间对这段区域的修改也直接反映用户空间。正因为有这样的映射关系，我们就不需要再用户态和内核态之间拷贝数据，提高了数据传输的效率。
- Netty的Zero-copy完全是再用户态的，它的零拷贝更多的是偏向于优化数据操作这样的概念。怎么说呢，看下面它是如何完成零拷贝操作的就知道啦~

##传统意义的零拷贝
这种方式需要四次数据拷贝和四次上下文切换：
1. 数据从磁盘读取到内核的read buffer
2. 数据从内核缓冲区拷贝到用户缓冲区
3. 数据从用户缓冲区拷贝到内核的socket buffer
4. 数据从内核的socket buffer拷贝到网卡接口的缓冲区
明显上面的第二步和第三步是没有必要的，通过java的FileChannel.transferTo方法，可以避免上面两次多余的拷贝（当然这需要底层操作系统支持）
1. 调用transferTo，数据从文件由DMA引擎拷贝到内核read buffer
2. 接着DMA从内核read buffer将数据拷贝到网卡接口buffer

##Netty中的零拷贝和它的几种设计
Netty中也用到了FileChannel.transferTo方法，所以Netty的零拷贝也包括上面的操作系统级别的零拷贝。除此之外！！！再ByteBuf的实现上，Netty也提供了零拷贝的一些实现。具体说来它有这几种实现零拷贝的技术：
- Netty提供了CompisiteByteBuf类，它可以将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf之间的拷贝。
- 通过wrap操作，我们可以将byte[]数组、ByteBuf、ByteBuffer等包装成一个Netty ByteBuff对象，进而避免了拷贝操作。
- ByteBuf支持slice操作，因此可以将BtyeBuf分解为多个共享同一个存储区域的ByteBuf，避免了内存的拷贝。
- 通过FileRegion包装的FileChannel.tranferTo实现文件传输，可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。

##单独分析几种拷贝方式
###CompisiteByteBuf
假设我们有一份协议数据, 它由头部和消息体组成, 而头部和消息体是分别存放在两个 ByteBuf 中的, 我们在代码处理中, 通常希望将 header 和 body 合并为一个 ByteBuf, 方便处理, 那么通常的做法是将 header 和 body 都拷贝到了新的 allBuf 中了, 这无形中增加了两次额外的数据拷贝操作了。CompisiteByteBuf的做法是将 header 与 body 合并为一个逻辑上的 ByteBuf。虽然看起来 CompositeByteBuf 是由两个 ByteBuf 组合而成的, 不过在 CompositeByteBuf 内部, 这两个 ByteBuf 都是单独存在的, CompositeByteBuf 只是逻辑上是一个整体。
###wrap
例如我们有一个 byte 数组, 我们希望将它转换为一个 ByteBuf 对象, 以便于后续的操作, 那么传统的做法是将此 byte 数组拷贝到 ByteBuf 中,这样的方式也是有一个额外的拷贝操作的, 我们可以使用 Unpooled 的相关方法, 包装这个 byte 数组, 生成一个新的 ByteBuf 实例, 而不需要进行拷贝操作.
####slice 操作
slice 操作和 wrap 操作刚好相反, Unpooled.wrappedBuffer 可以将多个 ByteBuf 合并为一个, 而 slice 操作可以将一个 ByteBuf 切片 为多个共享一个存储区域的 ByteBuf 对象。说白了就是用slice方式可以访问一个整体的某一部分，而不需要单独进行切分再访问。
###FileRegion
它是依赖于Java NIO FileChannel.transfer 的零拷贝功能。说白了就是直接拷贝，将文件内容直接写入channel或者直接从channel读到文件中，而不需要中间的一个buffer来进行内存缓冲