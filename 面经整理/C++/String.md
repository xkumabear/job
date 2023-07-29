1. 实现string拷贝构造函数
2. sizeof和strlen的区别
   1. sizeof()是运算符，strlen()是库函数
   2. sizeof()在编译时计算好了，strlen()在运行时计算
   3. sizeof()计算出对象使用的最大字节数，strlen()计算字符串的实际长度
   4. sizeof()的参数类型多样化（数组，指针，对象，函数都可以），strlen()的参数必须是字符型指针（传入数组时自动退化为指针）