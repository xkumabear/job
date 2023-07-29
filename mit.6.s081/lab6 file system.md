## 文件系统

#### XV6文件系统的特点

#### 文件系统背后的机制

​	文件系统对于硬件的抽象是如何实现的

​	crash safety。有可能在文件系统的操作过程中，计算机崩溃了，在重启之后你的文件系统仍然能保持完好，文件系统的数据仍然存在，并且你可以继续使用你的大部分文件。如果文件系统操作过程中计算机崩溃了，然后你重启之后文件系统不存在了或者磁盘上的数据变了，那么崩溃的将会是你。

​	如何在磁盘上排布文件系统。例如目录和文件，它们都需要以某种形式在磁盘上存在，这样当你重启计算机时，所有的数据都能恢复。所以在磁盘上有一些数据结构表示了文件系统的结构和内容。在XV6中，使用的数据结构非常简单，因为XV6是专门为教学目的创建的。真实的文件系统通常会更加复杂。

​	文件系统所在的硬件设备通常都较慢，比如说向一个SSD磁盘写数据将会是毫秒级别的操作，而在一个毫秒内，计算机可以做大量的工作，所以尽量避免写磁盘很重要，我们将在几个地方看到提升性能的代码。比如说，所有的文件系统都有buffer cache或者叫block cache。同时这里会有更多的并发，比如说你正在查找文件路径名，这是一个多次交互的操作，首先要找到文件结构，然后查找一个目录的文件名，之后再去查找下一个目录等等。你会期望当一个进程在做路径名查找时，另一个进程可以并行的运行。这样的并行运行在文件系统中将会是一个大的话题。



文件系统中核心的数据结构就是inode和file descriptor。后者主要与用户进程进行交互。

尽管文件系统的API很相近并且内部实现可能非常不一样。但是很多文件系统都有类似的结构。因为文件系统还挺复杂的，所以最好按照分层的方式进行理解。可以这样看：

- 在最底层是磁盘，也就是一些实际保存数据的存储设备，正是这些设备提供了持久化存储。
- 在这之上是buffer cache或者说block cache，这些cache可以避免频繁的读写磁盘。这里我们将磁盘中的数据保存在了内存中。
- 为了保证持久性，再往上通常会有一个logging层。许多文件系统都有某种形式的logging，我们下节课会讨论这部分内容，所以今天我就跳过它的介绍。
- 在logging层之上，XV6有inode cache，这主要是为了同步（synchronization），我们稍后会介绍。inode通常小于一个disk block，所以多个inode通常会打包存储在一个disk block中。为了向单个inode提供同步操作，XV6维护了inode cache。
- 再往上就是inode本身了。它实现了read/write。
- 再往上，就是文件名，和文件描述符操作。

这里有些术语有点让人困惑，它们是sectors和blocks。

- sector通常是磁盘驱动可以读写的最小单元，它过去通常是512字节。
- block通常是操作系统或者文件系统视角的数据。它由文件系统定义，在XV6中它是1024字节。所以XV6中一个block对应两个sector。通常来说一个block对应了一个或者多个sector。

![img](https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MRhBFGrbi6iZo-luZ0H%2F-MRhy7Cd_NccmqjQhB8S%2Fimage.png?alt=media&token=7cc0655d-df6a-4d7b-9283-a4953d3f105a)

文件系统的工作就是将所有的数据结构以一种能够在重启之后重新构建文件系统的方式，存放在磁盘上。虽然有不同的方式，但是XV6使用了一种非常简单，但是还挺常见的布局结构。

通常来说：

- block0要么没有用，要么被用作boot sector来启动操作系统。
- block1通常被称为super block，它描述了文件系统。它可能包含磁盘上有多少个block共同构成了文件系统这样的信息。我们之后会看到XV6在里面会存更多的信息，你可以通过block1构造出大部分的文件系统信息。
- 在XV6中，log从block2开始，到block32结束。实际上log的大小可能不同，这里在super block中会定义log就是30个block。
- 接下来在block32到block45之间，XV6存储了inode。我之前说过多个inode会打包存在一个block中，一个inode是64字节。
- 之后是bitmap block，这是我们构建文件系统的默认方法，它只占据一个block。它记录了数据block是否空闲。
- 之后就全是数据block了，数据block存储了文件的内容和目录的内容

在xv6中一个block 为1024 字节 一个inode 64占64字节

inode究竟是什么？这是一个64字节的数据结构。

- 通常来说它有一个type字段，表明inode是文件还是目录。
- nlink字段，也就是link计数器，用来跟踪究竟有多少文件名指向了当前的inode。
- size字段，表明了文件数据有多少个字节。
- 不同文件系统中的表达方式可能不一样，不过在XV6中接下来是一些block的编号，例如编号0，编号1，等等。XV6的inode中总共有12个block编号。这些被称为direct block number。这12个block编号指向了构成文件的前12个block。举个例子，如果文件只有2个字节，那么只会有一个block编号0，它包含的数字是磁盘上文件前2个字节的block的位置。
- 之后还有一个indirect block number，它对应了磁盘上一个block，这个block包含了256个block number，这256个block number包含了文件的数据。所以inode中block number 0到block number 11都是direct block number，而block number 12保存的indirect block number指向了另一个block。

目录

大部分Unix文件系统有趣的点在于，一个目录本质上是一个文件加上一些文件系统能够理解的结构。在XV6中，这里的结构极其简单。每一个目录包含了directory entries，每一条entry都有固定的格式：

- 前2个字节包含了目录中文件或者子目录的inode编号，
- 接下来的14个字节包含了文件或者子目录名

​		所以每个entry总共是16个字节。

对于block cache代码的介绍。这里有几件事情需要注意：

- 首先在内存中，对于一个block只能有一份缓存。这是block cache必须维护的特性。
- 其次，这里使用了与之前的spinlock略微不同的sleep lock。与spinlock不同的是，可以在I/O操作的过程中持有sleep lock。
- 第三，它采用了LRU作为cache替换策略。
- 第四，它有两层锁。第一层锁用来保护buffer cache的内部数据；第二层锁也就是sleep lock用来保护单个block的cache。



block cache的实现，这对于性能来说是至关重要的，因为读写磁盘是代价较高的操作，可能要消耗数百毫秒，而block cache确保了如果我们最近从磁盘读取了一个block，那么我们将不会再从磁盘读取相同的block。
