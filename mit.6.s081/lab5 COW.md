### **page fault异常**

下面三种情况，内核会产生page fault：

1. 当进程使用的**虚拟地址并没有在页表中映射**
2. 页表映射的**pte没有PTE_V标志位**
3. 或者**权限不符合**（读、写、执行不满足PTE_R、PTE_W、PTE_X等pte标志位）

内核将page-fault分为三类：

1. load page fault：load地址翻译出错
2. store page fault：store指令地址翻译出错
3. instruction page fault：pc寄存器地址翻译出错

stcause寄存器标志发生page fault的类型；stval寄存器表示发生page fault的地址 sepc寄存器表示当前发生page fault的指令地址。

原理就是通过引入一个中间层的方式解决相应的问题

### **COW**

父子进程共用一个页表，只有在父子任意一个进程需要写一个页的时候，因为PTE_W清除了。导致page fault，然后为产生page fault的进程分配页面，把共用的页面拷贝过去。

### **Lazy Allocation**

进程在通过sbrk或者alloc等函数申请内存的时候，可以先在进程描述符中更新sz，即进程的大小，而不实际分配内存，只有在真正使用这块内存的时候，首先会产生page fault，然后再真正分配一个页面。

### **Demand Paging**

这个和Lazy Allocation基本差不多，不过Lazy针对进程申请内存而言的。Demand paging针对exec加载进程内存空间的时候的代码段、数据段等而言。从而消除程序很大而导致的加载慢的问题。

### **Paging to Disk**

当物理内存不够用的时候，可以把进程的页表暂时写到磁盘中，然后把这一部分的页表设置为无效invaild，当访问这部分内存的时候，导致page fault，然后再从磁盘中拿到内存中，设置好PTE_V。