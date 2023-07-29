- 设计模式，观察者模式

- 单例设计模式：
  - 保证一个类仅有一个实例，并提供一个访问它的全局访问点，该实例被所有程序模块共享。
  
  - 定义一个单例类：
    1. 私有化它的构造函数，以防止外界创建单例类的对象；
    2. 使用类的私有静态指针变量指向类的唯一实例；
    3. 使用一个公有的静态方法获取该实例。
    
  - 最安全的单例设计模式：
  
    - ```c++
      class Singleton{
      private:
      	singleton(){}
          ~singleton(){}
          singleton(const singleton& s) = delete;
          singleton& operator=(const singleton& s) = delete;
      public:
          static singleton& instance(){
              static singleton sg;
              return sg;
          }
      }
      每次使用该方法 不需要判断的开销
      不会产生内存泄漏，对象不是指针；
      多线程下保证只会产生一次实例
      虽然因为 构造函数初始化过程不是线程安全的，但是在vs2010后已经解决。
       第一步：内存分配
       第二步：初始化成员变量
       由于多线程的关系，可能当我们在分配内存好了以后，还没来得急初始化成员变量，就
       进行线程切换，另外一个线程拿到所有权后，由于内存已经分配好了，但是变量初始化                        
      
      ```
  
      