sort 函数可传入一个 static bool cmp（ T a ， T b）{}作为比较函数，值得注意的是，函数需为static ，且其中用到的其他函数也需要以 static开头。

lower_bound( )和upper_bound( )都是利用[二分查找](https://so.csdn.net/so/search?q=二分查找&spm=1001.2101.3001.7020)的方法**在一个排好序的数组**中进行查找的。

lower_bound( begin,end,num)：从数组的begin位置到end-1位置二分查找第一个大于或等于num的数字，找到返回该数字的地址，不存在则返回end。**通过返回的地址减去起始地址begin,得到找到数字在数组中的下标。**

upper_bound( begin,end,num)：从数组的begin位置到end-1位置二分查找第一个大于num的数字，找到返回该数字的地址，不存在则返回end。**通过返回的地址减去起始地址begin,得到找到数字在数组中的下标。**



C++ iota()函数 对一个范围数据进行赋值：

```cpp
 iota(ids, ids + n, 0);
```