# STL解读

侯捷视频

## 2.STL六大部件

- 容器
- 分配器
- 算法
- 迭代器
- 适配器
- 仿函数

![image-20200313145801573](STL%E8%A7%A3%E8%AF%BB.assets/image-20200313145801573.png)



- 前闭后开

  ```c++
  vector<int> v;
  for(auto ite = v.begin();ite!=v.end();ite++){
    cout << *ite << endl;
  }
  
  for(auto elem : v){
    cout << elem << endl;
  }
  for(auto& elem : v){
    elem*=3;
  }
  ```

  

## 3.容器分类与测试

![image-20200313151737780](STL%E8%A7%A3%E8%AF%BB.assets/image-20200313151737780.png)

### array

```c++
array<long,onst int size> c;

c[0] = 100;
c.begin();
c.end();
c.size();
c.front();
c.back();
```

### vector

```c++
vector<int> v;
v.push_back(100); //capacity2倍增长，1，2，4，8

vector<int>(int size,int initValue);
v[0] = 100;

v.size();
v.capacity();

```



### deque

### list

### forward_list

### set/multiset

### map/multimap

### unordered set/multiset

### undored map/multimap



## STL深度剖析

### list

底层实现：双向循环链表，多了一个空元素，为了满足end() 

空间不连续，迭代器重新包装成一个类，通过next指针找到下一个元素，同理，--通过prev指针找到前一个元素

![image-20200313164200387](STL%E8%A7%A3%E8%AF%BB.assets/image-20200313164200387.png)



### vector

底层数组，capacity两倍增长

![image-20200313164234452](STL%E8%A7%A3%E8%AF%BB.assets/image-20200313164234452.png)



### array

一个固定数组

### forward_list

单向链表

### deque && stack && queue 

- deque

  ![image-20200313165214071](STL%E8%A7%A3%E8%AF%BB.assets/image-20200313165214071.png)

  - **map，一个vector，里面存放着指针，因此map本身类型为T** （最小为8，最大为所需节点数+2，所需节点数为num_elements/buffer_size()+1，buffer_size()为512/size(元素类型)）**
  - 迭代器 自写的类，包含四个成员，T* frist，T* last，T* cur，T** node，分别指向buffer的头，尾，当前，和该buffer在map中的位置
  - 一个deque里面有四个成员，iterator start，iterator finish，map_pointer map，size_type map_size
  - Gun2.9可以指定buffer的元素大小，默认设置是512/size(元素)，如果一个元素大于512字节，那默认buffer的元素为1；G4.9不可以指定，直接使用默认
  - **map这个vector扩充时，会将原有元素搬到中间位置，为了让前后均可以扩增**

- queue

  - template< class  T , class Sequence = deque< T > >
  - 因此其实就是deque的一个适配器

- stack

  - 与queue一样，其实也是deque的一个适配器

- 当然其实也可以以list< T >为底层容器

- queue和stack不提供迭代器，不允许遍历

### 容器rb_tree

- 红黑树是平衡二叉搜索树
- 提供迭代器，可以遍历（排序顺序，其实就是中序）

- set/mutilset/map/mutilmap底层都是rb_tree

### 容器hashtable<考点>

- 解决冲突

  - 开放定址
    - 线性探测 +1 ， +2， +3
    - 平方探测 +1方，+2方，+3方
    - 伪随机探测
  - 拉链（实际使用这种）
  - 再散列：多个哈希方法，一个冲突，第二个

- 篮子数刚开始是53，当元素>53时，扩展篮子数，开展后是53的2倍附近的质数，97；然后进行重新哈希

  - 质数已经写好了，在一个数组里放着

- unorder_set/unorder_map都是通过hashtable实现的

  

## 仿函数Functor

```c++
//使用排序函数
bool myfunc(int x ,int y){return x<y;}
sort(myvec.bengin(),myvec.end(),myfunc);

//使用仿函数，未融入STL
struct mycompare {
	bool operator()(int x,int y){return x<y;}
}myobj;

sort(vec.begin(),vec.end(),myobj);

//融入STL，需要继承binary_function(两操作数),unary_function(一操作数),即有了可适配的条件，有了typedef，不占类空间，但可以融入STL，即可以被适配器修改，将参数和返回值的类型重命名，以让适配器获得
template<class Arg,class Result>
  class unary_function{
  	typedef Arg argument_type;
  	typedef Result result_type;
	}
template<class Arg1,class Arg2,class Result>
  class binary_function{
  	typedef Arg1 first_argument_type;
  	typedef Arg2 second_argument_type;
  	typedef Result result_type;
	}

//greater融入STL,继承binary_function(两操作数)
template<class T>
  struct greater:public binary_function<T,T,bool>{
    bool operator()(const T& x, const T& y) const{
      return x>y;
    }
  }

```

## 适配器Adapters

存在很多Adapters:

​	Iterator Adapters

​	Container Adapters

​	Functor Adapters

有两种方式可以使用其他类的函数，继承，和组合，适配器都会使用组合，即适配器里包含Iterator,Container,Functor等

- Container Adapters

  ```c++
  //stack,queue内置了deque
  ```

- Functor Adapters

  ```c++
  //binder2nd通过参数，获取内置的仿函数的第二个参数
  ```

- Iterator Adapters

  ```c++
  //reverse_iterator:  比如rbegin(),rend()
  reverse_iterator
    rbegin(){
    return reverse_iterator(end());
  }
  reverse_iterator
    rend(){
    return reverse_iterator(begin());
  }
  ```







