1. lambda表达式的原理
   1. 编译器会把一个Lambda表达式生成一个闭包类的闭包（匿名对象），并在类中**重载函数调用运算符**，实现了一个`operator()`方法。
   
2. lambda表达式，值捕获，引用捕获

3. bind绑定器 和 function 机制；

   1. ```c++
      template<typename Compare, typename T>
      _mybind1st<Compare, T> mybind1st(Compare comp, const T &val) { //绑定器返回值\_mybind1st为一元函数对象
          return _mybind1st<Compare, T>(comp, val);
      }
      ```

   2. bind （函数适配器） 是对上面二元的泛化，最多可以绑定29元

   3. bind 绑定普通函数，第一个参数是函数对象，后面是绑定参数，和占位符

   4. bind 绑定类非静态成员函数，第一个参数是函数对象，第二个是匿名函数对象本身，后面是绑定参数，和占位符

      

4. std::function <T> 是c++中的函数指针的一种实现，常用来实现回调功能。

5. 为什么有函数指针还需要std::function？

   1. 单纯的函数指针，没有携带或捕捉上下文的能力；
   2. std::function 封装了函数指针和它的上下文；
   3. 函数指针可用于回调功能，函数对象也可用于回调功能，lambda表达式也可用于回调功能，甚至bind绑定适配后的成员函数也可用于回调功能，那么在不确定的情况下，通过function机制这样的泛型机制统一表示，就会很方便。

6. 回调和闭包中可以通过 function来实现吗
