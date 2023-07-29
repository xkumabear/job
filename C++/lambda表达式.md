谓词：

闭包：lambda表达式创建的运行期对象。（运行期）

闭包类：实例化闭包的类 （编译期）

lambda表达式 （编译期）

对于每个lambda 都会触发编译器生成独一无二的闭包类 ，其中函数体成为闭包类的成员函数

捕获模式：

默认的两种捕获模式：按引用或按值

按引用：可能会导致空悬引用

按值:在类内，捕获的是this的值副本，所以其闭包的存活周期和其含有的this指针绑定在一起的 ，所以也会产生空悬指针。 (误导了，认为它是自洽的)

解决办法，提前使用局部变量保存成员变量

广义按值捕获：

```c++
[d = d](int val){ return;}
```

注：在c++11中无法移动构造一个对象到闭包

可以使用c++14的初始化捕获将对象移入闭包

或者手工实习类，和bind去模拟初始化捕获；



c++14 实现完美转发lambda表达式以及变长参数：

```c++
auto f = [](auto&&... params){ 
	return func(normalize(std::forward<decltype(params)>(params)...) )
	//对 auto&& 型别的形参使用decltype 来判断 std::forward<t>()的类型
}
```

尽量使用lambda 而不是 bind 因为lambda具有更高的可读性，

![image-20230228163018344](C:\Users\qq130\AppData\Roaming\Typora\typora-user-images\image-20230228163018344.png)

##### Lamdba表达式适用场景

sort  find_if `for_each`  `remove_if`



#### 尾序返回值的好处：

在指定返回值型别时可以使用函数的形参。因为如果传统的先序语法 会认为该形参还未被定义；