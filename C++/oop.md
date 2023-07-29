虚函数表内存空间

![image-20230228192458349](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20230228192458349.png)

一个类中有虚函数，那么该类就有一个虚函数表。且一个类只能有一个虚函数表。在编译时，一个类的虚函数表就确定了，放在了只读数据段中。

![在这里插入图片描述](https://img-blog.csdnimg.cn/cacc3d8083624b918ea9ea555ae2f8ce.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57K-6Ie055qE54GwKD5fPCk=,size_20,color_FFFFFF,t_70,g_se,x_16)

1. 构造函数：
2. 析构函数：
   1. 析构函数不是完成对象的销毁，局部对象销毁工作是由编译器完成的。而**对象在销毁时会自动调用析构函数，完成类的一些资源清理工作。**
   2. 对类的资源进行处理善后
   3. 其中**内置资源不需要清理**，***最后系统直接将其内存回收即可***
   4. 需要的是对自定义的资源对象的指针进行回收；
   5. 析构函数不销毁对象，只负责善后对象所拥有的指针。
   6. 所以引出free new 对象的问题；