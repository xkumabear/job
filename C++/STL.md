#  STL

六大组件: **容器、适配器、空间配置器、迭代器、仿函数、算法**



三类容器：

- 顺序容器、关联容器

- 顺序容器：vector list deque forward_list array string

- 关联容器：
  - 有序容器：
  - 无序容器：
- 所有容器都提供的操作：
- 只有顺序容器接收大小参数，关联容器不支持

#### 顺序容器

1. 所有顺序容器都提供的操作：
   - 容器定义和初始化： 支持默认构造，拷贝构造，拷贝赋值运算、列表初始化 、迭代器范围拷贝初始化、和指定数量和初始值初始化
   - swap ： 只交换了两个容器的内部数据结构（array除外）。O(1)的时间复杂度； 迭代器虽然不会失效，但是却指向了不同的容器。
   - 大小操作：size  empty  max_size
   - 关系运算符：所有容器都支持 == !=  有序和顺序支持 大小与 但要求类型严格相等。与string一样是逐个比较的。
     - 容器使用元素的关系运算符进行比较，所以只有定义了关系运算符的才可以比较。 <
   - 添加删除操作：
     - vector和string 不支持push_front 和 emplace_front
     - push_back&emplace_back
     - push_front&emplace_front
     - insert( p ,v) : p为迭代器，在p前插入值为v的元素 //返回插入的第一个位置的迭代器，或原迭代器；可以来完成循环头插；
       - insert(p,n,t) ; insert( p ,b , e); insert( p , {li})
     - emplace(p,args);：emplace 系列会使用传入值（参数），调用构造函数直接在管理内存空间 直接创建对象；
     - 向vector string deque 插入元素会使所有迭代器失效；因为申请空间导致元素地址移动；
2. 某个容器独有的操作：



顺序容器如何构造元素的：

- 顺序容器调用了元素类型的默认构造函数
- 若没有默认构造则需要提供元素初始化器。



##### 迭代器



