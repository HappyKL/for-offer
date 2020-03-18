## 迭代器失效

```c++
//对于vector,deque等序列容器来说，使用erase(iter)会造成后面的迭代器失效,erase函数的返回值会返回一个有效的iter

//对于map，set这些容器来说，使用erase(iter)，只会使当前迭代器失效，不会返回有效迭代器，，因此在调用erase之前，需要记录下一个元素的迭代器
erase(iter++);//先传参数到erase里，然后将iter+1，然后执行erase函数

//对于list来说，使用erase(iter)，只会使当前迭代器失效，也会返回一个有效的iter，也可以提前记录下一个元素的迭代器

```

