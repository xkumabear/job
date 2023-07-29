`exec`是创建地址空间的用户部分的系统调用。它使用一个存储在文件系统中的文件初始化地址空间的用户部分



1. 打开程序的二进制。读取ELF头。每个`proghdr`描述程序中必须加载到内存中的一节
2. 校验文件的完整性
3. 分配新页表，为每个elf段分配内存，加载程序段。
4. 分配并初始化用户栈，分配一个栈页面。伪返回程序计数器入栈 ，argc argv 指针 ，栈的正下方 为不可访问页面。防止越界访问，大参数问题，
5. 加载elf文件中的字节到文件指定的地址中，因此有两个问题，用户空间可能会访问到内核空间地址，xv6通过地址检查，权限设置来解决
   1. 当用户地址空间包含内核时，（虽然在用户模式下没有访问权限）可能会造成内核数据被替换的情况。
   2. 目前不包含内核地址空间。

函数调用的过程：

```c
void
backtrace(void)
{
  printf("backtrace: \n");
  uint64  pfp = r_fp();
  uint64 battel = PGROUNDUP(pfp); //栈底   该栈是从高地址为底，低地址为顶
  uint64 top =PGROUNDDOWN(pfp); //栈顶   该栈是从高地址为底，低地址为顶
  uint64 reAddr;
  if( pfp < top ){
    printf("  error!\n");  //pfp不会比当前top小，该步可省略！
    return;
  }
   
  while( pfp < battel ){ //找到栈底为止
    reAddr = *((uint64*)(pfp-8)); //调用的返回指令地址
    pfp = *((uint64*)(pfp-16));  //之前调用者保存的堆栈
    printf("%p\n",reAddr);
  }

}
```

