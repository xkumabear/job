# 操作系统的隔离性与防御性

### 操作系统隔离性 isolation

需要在不同的应用程序之间有强隔离性。 一个程序不应该影响另一个程序

会希望操作系统仍然能较好的处理异常情况。所以，你也需要在应用程序和操作系统之间有强隔离性。

总结 ： 应用程序不应影响其他应用程序。

​			操作系统不应该被应用程序影响。



如果没有隔离性会发生什么：

​	如当一个应用程序中存在死循环，独占cpu资源，没有一个操作系统杀掉这个进程 其他程序无法执行  即无法 多进程同分时复用

​	如从内存的角度，shell、echo的应用程序运行时代码、数据同时存在内存中，此时物理内存同时被shell 和 echo 使用，其间没有边界，此时一个程序的数据可能会覆盖另一个程序 再内存中的数据。  即  内存上不是隔离的



使用操作系统的主要原因就是为了实现multiplexing和内存隔离

接口通过抽象硬件资源，从而使得提供强隔离性成为可能 ，实现资源的强隔离



应用程序不能直接与CPU交互，只能与进程交互。操作系统内核会完成不同进程在CPU上的切换。所以，操作系统不是直接将CPU提供给应用程序，而是向应用程序提供“进程”，进程抽象了CPU，这样操作系统才能在多个应用程序之间复用一个或者多个CPU。



### 操作系统防御性（Defensive）

操作系统需要能够应对恶意的应用程序。

另一个需要考虑的是，应用程序不能够打破对它的隔离。

### 硬件对于强隔离的支持

硬件对于强隔离的支持包括了：user/kernle mode和虚拟内存。

​       为了支持user/kernel mode，处理器会有两种操作模式，

**第一种是user mode**

**第二种是kernel mode**

当运行在kernel mode时，CPU可以运行特定权限的指令；

当运行在user mode时，CPU只能运行普通权限的指令（unprivileged instructions）。



普通权限的指令都是一些熟悉的指令，例如将两个寄存器相加的指令ADD、将两个寄存器相减的指令SUB、跳转指令JRC、BRANCH指令等等

特殊权限指令主要是一些直接操纵硬件的指令和设置保护的指令，例如设置page table寄存器、关闭时钟中断。

在处理器上有各种各样的状态，操作系统会使用这些状态，但是只能通过特殊权限指令来变更这些状态。



提问：

1并且怎么知道当前是不是kernel mode？是有什么标志位吗？

在处理器里面有一个flag。在处理器的一个bit，当它为1的时候是user mode，当它为0时是kernel mode。当处理器在解析指令时，如果指令是特殊权限指令，并且该bit被设置为1，处理器会拒绝执行这条指令，就像在运算时不能除以0一样。

2 那BIOS呢？BIOS会在操作系统之前运行还是之后？

Frans教授：BIOS是一段计算机自带的代码，它会先启动，之后它会启动操作系统，所以BIOS需要是一段可被信任的代码，它最好是正确的，且不是恶意的。

3 **设置处理器中kernel mode的bit位的指令是一条特殊权限指令，那么一个用户程序怎么才能让内核执行任何内核指令？因为现在切换到kernel mode的指令都是一条特殊权限指令了，对于用户程序来说也没法修改那个bit位。**

用户程序会通过系统调用来切换到kernel mode。当用户程序执行系统调用，会通过ECALL触发一个**软中断**（software interrupt），软中断会查询操作系统预先设定的中断向量表，并执行中断向量表中包含的中断处理程序。中断处理程序在内核中，这样就完成了user mode到kernel mode的切换，并执行用户程序想要执行的特殊权限指令。



硬件对于支持强隔离性的**第二个特性**，基本上所有的CPU都支持**虚拟内存。**

处理器包含了page table，而page table将虚拟内存地址与物理内存地址做了对应。

每一个进程都会有自己独立的page table，这样的话，每一个进程只能访问出现在自己page table中的物理内存。

操作系统会设置page table，使得每一个进程都有不重合的物理内存，这样一个进程就不能访问其他进程的物理内存，因为其他进程的物理内存都不在它的page table中。一个进程甚至都不能随意编造一个内存地址，然后通过这个内存地址来访问其他进程的物理内存。这样就给了我们内存的强隔离性。

类似的，内核位于应用程序下方，假设是XV6，那么它也有自己的内存地址空间，并且与应用程序完全独立。



需要有一种方式能够让应用程序可以将控制权转移给内核（Entering Kernel）。

在RISC-V中，有一个专门的指令用来实现这个功能，叫做ECALL。ECALL接收一个数字参数，当一个用户程序想要将程序执行的控制权转移到内核，它只需要执行ECALL指令，并传入一个数字。这里的数字参数代表了应用程序想要调用的System Call。

ECALL会跳转到内核中一个特定，由内核控制的位置。每一次应用程序执行ECALL指令，应用程序都会通过这个接入点进入到内核中

用户空间和内核空间的界限是一个硬性的界限，用户不能直接调用fork，用户的应用程序执行系统调用的唯一方法就是通过这里的ECALL指令。

### 宏内核 vs 微内核 （Monolithic Kernel vs Micro Kernel）

通过系统调用或者说ECALL指令，将控制权从应用程序转到操作系统中。之后内核负责实现具体的功能并检查参数以确保不会被一些坏的参数所欺骗。所以内核有时候也被称为可被信任的计算空间（Trusted Computing Base），在一些安全的术语中也被称为TCB。

要被称为TCB，内核首先要是正确且没有Bug的。假设内核中有Bug，攻击者可能会利用那个Bug，并将这个Bug转变成漏洞，这个漏洞使得攻击者可以打破操作系统的隔离性并接管内核。所以内核真的是需要越少的Bug越好

宏内核

让整个操作系统代码都运行在kernel mode。大多数的Unix操作系统实现都运行在kernel mode。比如，XV6中，所有的操作系统服务都在kernel mode中，这种形式被称为Monolithic Kernel Design（[宏内核](https://en.wikipedia.org/wiki/Monolithic_kernel)）。

缺点：因为系统代码体量大，bug会更多。每一个bug都有可能变成为严重漏洞

优点:  因为各子系统的紧密集成，提供了良好的性能。

微内核

关注点是减少内核中的代码，它被称为Micro Kernel Design（[微内核](https://en.wikipedia.org/wiki/Microkernel)）。在这种模式下，希望在kernel mode中运行尽可能少的代码。所以这种设计下还是有内核，但是内核只有非常少的几个模块，例如，内核通常会有一些IPC的实现或者是Message passing；非常少的虚拟内存的支持，可能只支持了page table；以及分时复用CPU的一些支持。

优点：内核中的代码的数量较小，更少的代码意味着更少的Bug。

缺点：

1 典型的通过消息来实现传统的系统调用。在user/kernel mode反复跳转带来的性能损耗。

2 在一个类似宏内核的紧耦合系统，各个组成部分，例如文件系统和虚拟内存系统，可以很容易的共享page cache。而在微内核中，每个部分之间都很好的隔离开了，这种共享更难实现。进而导致更难在微内核中得到更高的性能。





vx6:

**一个进程捆绑了两个设计想法：**

**- 一个是地址空间，给一个进程拥有一个内存的”假象“。**

**- 另一个是线程，给这个进程拥有一个私有的CPU的”假象“。**

一个进程由一个地址空间和一个线程组成。在实际的操作系统中，一个进程可能有多个线程来利用多个cpu。



实验一：写一个用于追踪目标系统调用的 系统调用函数

实验二： 写一个用查看当前系统资源信息的系统调用函数

实验总结：

系统调用的实现过程

首先在用户层声明 系统调用函数

通过桩代码 将声明的函数与 真实的系统调用号相连

usertrap 调用 syscall

syscall 中 调用系统调用号 对应 系统调用函数

通过用户应用空间获取参数

![image-20221116162558022](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20221116162558022.png)

![image-20221116162817961](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20221116162817961.png)

![image-20221116162648774](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20221116162648774.png)