# Trap机制

用户空间和内核空间的切换：

- 程序执行系统调用
- 程序出现了类似page fault、运算时除以0的错误
- 一个设备触发了中断使得当前程序运行需要响应内核设备驱动

都会发生这样的切换

用户空间和内核空间的切换通常被称为trap，而trap涉及了许多小心的设计和重要的细节，这些细节对于实现安全隔离和性能来说非常重要。因为很多应用程序，要么因为系统调用，要么因为page fault，都会频繁的切换到内核中。所以，trap机制要尽可能的简单，这一点非常重要。

硬件的状态将会非常重要，因为我们很多的工作都是将硬件从适合运行用户应用程序的状态，改变到适合运行内核代码的状态。

有一个特殊的寄存器我想讨论一下，也就是mode标志位。这里的mode表明当前是user mode还是supervisor mode。

supervisor mode可以控制什么？

1. 可以读写控制寄存器了
2. supervisor mode可以做的是，它可以使用PTE_U标志位为0的PTE。

特别指出的是，supervisor mode中的代码并不能读写任意物理地址。在supervisor mode中，就像普通的用户代码一样，也需要通过page table来访问内存。如果一个虚拟地址并不在当前由SATP指向的page table中，又或者SATP指向的page table中PTE_U=1，那么supervisor mode不能使用那个地址。









###  **RISC-V的trap机制**

每个RISC-V的CPU都有一些控制寄存器，内核可以通过写寄存器告诉CPU怎么处理trap，内核也可以通过读相应的控制寄存器来判断发生了哪种trap。

- `stvec(supervisor trap vector)`：内核将trap处理程序的地址写到此寄存器，RISC-V就会跳转到此处来处理trap
- `sepc(supervisor exception program counter)`：当trap发生时，把pc寄存器的值保存到此处，（然后pc寄存器就会被stvec覆盖掉），`stret(supervisor trap return)`指令就会把sepc寄存器的值拷贝到pc寄存器，从而从trap中恢复。内核可以通过写sepc寄存器的值来控制pc跳转到何处。
- `scause(supervisor cause)`：RISC-V给这个寄存器赋值，通过这个取值来区分不同的trap的原因
- `sscratch`：内核设置一个值，在trap开始阶段会用到
- `sstatus`：这个控制寄存器中的SIE位控制着是否允许设备中断。如果内核清除了SIE位，内核将会屏蔽掉设备中断，直到SIE位被设置。SPP位表示trap来自于内核还是用户空间，并根据trap哪里，设置`sret`寄存器的取值，从而返回到trap之前的相应位置。

多核芯片上的没和CPU都有上述的这些寄存器，同一时间可能有多个CPU在处理trap。即每个CPU是独立的。

当发生trap的时候，RISC-V的硬件会做下面的处理：

1. 如果trap是设备中断，并且SIE位清零，就什么也不做
2. 清除`sstatus`寄存器中的SIE位，即不允许设备中断
3. 把pc寄存器的值拷贝到sepc寄存器，即保存pc寄存器
4. 保存`sstauts`寄存器中的SPP位，来保存当前模式（用户态还是内核态），trap发生于内核还是用户空间
5. 通过设置`scause`寄存器来反映trap的原因
6. 将模式设置为supervisor，即通过设置`sstatus`寄存器的SPP为内核态
7. 将stvec寄存器的值拷贝给pc，即将执行流切换到trap的处理向量上
8. 根据新的pc指针继续执行，完成trap处理

**总结一下：1、关闭设备中断；2、保存寄存器；3、设置寄存器使得进入内核，执行新的指令流**

### **用户空间trap**

xv6对于用户空间的trap以及内核空间的trap的处理方式是不同的。

需要处理的用户空间的trap包括syscall，以及非法的情况(地址访问越界)，或者设备中断。总体来说，用户控件的trap处理流程是：uservec->usertrap->trap处理程序->usertrapret->userret。

在xv6中一个主要的限制就是在发生trap的时候，不能立即切换页表。这意味着stvec寄存器指向的trap处理程序必须得有在用户进程空间的映射（因为在这之后才会切换到内核页表）。同时trap的处理程序需要在内核中执行，因此stvec也需要能够映射内核的页表地址。

因此xv6引入了一个trampoline页，stvec寄存器指向了在这一页中的trap处理程序uservec。trampoline页映射到了每一个xv6进程的相同地址处(TRAMPOLINE，进程地址空间最高处)。同时trampoline页也映射到了内核的TRAMPOLINE地址处，



32个寄存器的值都还是引起trap的进程所拥有的。这32个寄存器系需要被保存，以便后面的恢复。但是把这些数据保存到内存中，需要寄存器存放内存地址，但是已经没有空闲的通用寄存器了。好在还有`sscratch`寄存器。在uservec的最开始csrrw指令就会把a0寄存器的值和sscratch寄存器的值互换，





![image-20221123151635303](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20221123151635303.png)