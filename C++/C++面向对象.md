# C++ 面向对象 （上）

参考：侯捷 C++面向对象高级编程

- 目标
  - 单一类的设计 (基于对象 object based)
    - class without pointer members : Complex
    - class with pointer members : String
  - 学习Classes之间的关系 (面向对象 object oriented)
    - 继承 inheritance
    - 复合 composition
    - 委托 delegation

## 1.单一类的设计

- 个人理解：带指针成员的类，需要写拷贝构造，拷贝赋值和析构函数，而不带指针成员的类可以不写，默认的就可以

### 不带指针成员的类

以complex类为例

- 防范式声明

  任何一个头文件，都加以下内容

```c++
//complex.h
#ifndef __COMPLEX__
#define __COMPLEX__

//前置声明
class complex;
complex& __doapl(complex* ths, const complex& r);

//类声明
class complex{
  ...
};
//类定义
complex::function ...
  
#endif
```

- 声明

  ```c++
  class complex{						//class head
    public:									//class body
    	complex(double r = 0 , double i=0)  
        :re(r),im(i)       //初始列
        {}
    	complex& operator += (const complex&);
    	double real() const { return re; }
    	double imag() const { reutrn im; }
    private:
    	double re,im;
    	friend complex& __doapl(complex* ,const complex&)
  }
  ```

  涉及的知识点：

  - inline内联函数：函数在body里直接定义或者在body外使用inline关键字

  - 访问级别：public(外界可以被访问) / private(类自己访问)

  - 构造函数：默认参数，初始列

  - 构造函数可以有多个，可重载

  - 构造函数被放在private里

    ```c++
    //单例
    class A {
      public:
      	static A& getInstance();
      	setup() {...}
      private:
      	A();
      	A(const A& rhs);
      	...
    }
    
    A& A::getInstance(){
      static A a;
      return a;
    }
    ```

  - 常量成员函数：函数后加const

    不会改动数据内容，如果real函数不加const，如下

    ```c++
    class complex{					
      public:									
      	complex(double r = 0 , double i=0)  
          :re(r),im(i)      {}
      	double real(){ return re; }
      private:
      	double re,im;
      	
    }
    
    {
      const complex c1(1,2);
      cout << c1.real() << endl; //报错，因为real函数没有被标记为const，即可能会改变值，但c1是一个const类型的
    }
    ```

  - 参数传递：pass by value vs. pass by reference (to const)

    尽量pass by reference

    如果不希望改参数的值，要+const

  - 返回值传递：return by value vs. return by reference (to const)

    尽量return by reference

  - 友元friend：可以直接使用private

    ```c++
    complex& __doapl(complex* this,const complex& r){
      ths->re += r.re;
      ths->im += r.im;
      return *ths;
    }
    ```

    - 相同class的各个objects互为友元

      ```c++
      class complex{
        public:
        	complex (double r= 0,double i = 0)
            :re(r),im(i){}
        	int func(const complex& parm){
            return param.re + param.im;
          }
        private:
        	double re,im;
      }
      ```

- class body 外的各种定义

  - 全局函数

    - 无this

    - 操作符重载：比如+ ； 返回值不允许是reference，因为返回是一个local object(临时对象:typename())

      ```c++
      inline complex operator + ( const complex& x, const complex& y){
        return complex(real(x)+real(y),imag(x)+imag(y));   //临时对象:typename()
      }
      ```

  - 成员函数

    - 隐藏一个参数this
    - 操作符重载：比如+=；返回值是reference

- 完整编一个类（总结）

  - 防范声明

  - 类成员

  - 类构造函数（参数，默认值，参数类型是否reference，初值列，函数体）

  - 成员函数声明（参数及参数类型确定，是否inline，返回值及返回值类型确定，是否会修改数据内容标记为const）

  - 成员函数定义：是否inline

  - 全局函数：参数及参数类型确定，返回值及返回值类型（是否可以返回reference），比如<<的重载，必须全局函数设计，因为第一个参数是cout，而不是类

    ```c++
    #include<iostream.h>
    ostream& 
    operator << (ostream& os,const complex& x){
      return os << x;
    }
    ```

### 带指针成员的类

以string为例

- 框架

  ```c++
  #ifndef __MYSTRING__
  #define __MYSTRING__
  
  class string{
    
  }
  
  
  string::function(...)...
    
  Global-function(...)...
  
  #endif
  ```

- Big Three 三个特殊函数

  ```c++
  class string{
    public:
    	string(const char* cstr = 0);
    	string(const string& str);
    	string& operator=(const string& str);
    	~String();
    	char* get_c_str() const {return m_data;}
    private:
    	char* m_data;
  }
  
  ```

  - 构造和析构

    ```c++
    inline string::string(const char* cstr = 0){
      if(cstr){
        m_data = new char[strlen(cstr)+1];
        strcpy(m_data,cstr);
      }else{
        m_data = new char[1];
        *m_data = '\0';
      }
    }
    
    inline string::~string(){
      delete[] m_data;
    }
    ```

  - 拷贝构造copy ctor 

    需要自己拷贝指针成员的值

    ```c++
    inline string::string(const string& str){
      m_data = new char[strlen(str.m_data)+1];
      strcpy(m_data,str.m_data);
    }
    ```

  - 拷贝赋值函数 copy op=

    删除原先指针m_data的值（以防内存泄露），然后重新赋值

    ```c++
    inline string& string::operator=(const string& str){
      if(this == &str) return *this;
      delete[] m_data;
      m_data = new char[strlen(str.m_data)+1];
      strcpy(m_data,str.m_data);
      return *this;
    }
    ```

- 编写一个完整的类（总结）
  - 防卫声明
  - 成员变量确定
  - 构造函数，拷贝构造，拷贝赋值，析构
  - 成员函数声明
  - 成员函数定义
  - 全局函数，比如<<重载

## 2.stack和heap

- 个人理解：new malloc+构造  delete：析构+free

### 对象生命期

- stack object生命期：作用域结束之后
- static local object的生命期：整个程序结束之后
- global object的生命期：整个程序结束之后
- heap object：需要delete

### new和delete

- new
  - 分配内存 operator new() -> malloc
  - 类型强转
  - 构造函数
- delete
  - 析构函数
  - operator delete() -> free

- Array new 一定要搭配 array delete

  ![image-20200308170848023](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308170848023.png)

  

## 3.static && 单例 && 模板 && namespace

- static

  static函数中没有this指针

  static成员变量只有一份

  通过object或者class name调用

- ctors放在private

  ```c++
  class A {
    public:
    	static A& getInstance() {return a};
    	setup() {...}
    private:
    	A();
    	A(const A& rhs);
    	static A a;
    	...
  }
  
  ```

  

- 模板

  - 类模板
  - 函数模板

- namespace

   ```c++
  namespace std{
    ...
  }
  
  using namespace std;
  
  using namespace std::cout;
  
  //直接在代码里
  std::cout << 
  ```

  

## 4.组合/委托/继承

- 个人理解：复合是包含其他类实例；委托是包含其他类实例指针；析构函数：父类先调用，组合类再调用，子类再调用

- 组合：has-a

  ![image-20200308174751888](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308174751888.png)

  

  - 构造函数先调用Complent的default构造函数，然后才执行自己

    ```c++
    Container::Container(...):Component() {...};
    ```

    

  - 析构函数，先析构自己，然后才调用Component的析构函数

    ```c++
    Container::~Container(...){...~Component()};
    ```

    

- 委托 Composition by reference 

  这种方式很常用：Handle / Body  pimpl:pointor impletation.  接口与实现分离

  ![image-20200308180301815](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308180301815.png)

- 继承 is-a

  ![image-20200308180410694](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308180410694.png)

  - 构造函数，先调用父类，再调用子类
  - 析构：先调用子类，再调用父类



## 5.虚函数+继承

- 非虚函数：不希望子类重写
- 虚函数：希望子类重写
- 纯虚函数：子类必须重写

```c++
class Shape{
  public:
  	virtual void draw() const = 0;
  	virtual void error(const std::string& msg);
  	int objectID() const;
  
  class Rectangle:public Shape{...}
}
```

### Template Method

（Inheritance with virtual）

![image-20200308181921007](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308181921007.png)

对于写框架的程序会经常用到这种设计模式



## 6.继承+复合

- 构造：先父类，再组合类，再子类
- 析构：先子类，再组合类，再父类

## 7.委托+继承  

### 观察者模式 Observer

举例：相同的数据，四种不同展现形式，一种展现形式更改数据，其他展现形式需要相应变化

![image-20200308183715964](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308183715964.png)

![image-20200308183905443](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308183905443.png)



![image-20200308183806104](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308183806104.png)

### 复合模式 Composite

比如文件目录下有文件或目录

![image-20200308184354469](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308184354469.png)

### 原型模式 Protype

父类如何创建未来的子类实例？

子类的创建必须有一个静态成员是子类实例，因此创建时，自动调用构造函数，构造函数必须调用父类的addPrototype函数，从而添加到父类的prototypes容器里，然后父类就可以通过clone函数，new出子类的实例。子类必须重写clone函数

![image-20200308185754500](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200308185754500.png)

# C++面向对象（下）

## 1.目标

- 泛型编程 和 面向对象编程
- 深入面向对象之继承关系所形成的对象模型
  - 隐藏于底层的this指针
  - vptr(虚指针)
  - vtbl(虚表)
  - virtual mechanism(虚机制)
  - 虚函数造成的多态

## 2.转换函数

将类对象转化为其他类型

```c++
class Fraction{
  public:
  	Fraction(int num,int den=1):m_numerator(num),m_denominator(den){}
  	operator double() const{
      return (double)(m_numerator/m_denominator);
    }
  private:
  	int m_numerator;   //分子
  	int m_denominator;  //分母
}

Fraction f(3,5);
double d = 4+f; //调用operator double()将f转化为0.6
```



## 3.Non-explicit-one-argument ctor

- Non-explicit-one-argument ctor与转换函数不可并存

```c++
class Fraction{
  public:
  	Fraction(int num,int den=1):m_numerator(num),m_denominator(den){}
  	Fraction operator+(const Fraction& f){
      return Fraction(...);
    }
  private:
  	int m_numerator;   //分子
  	int m_denominator;  //分母
}

Fraction f(3,5);
Fraction d2=  f+4 ; //调用non-explicit ctor，将4转化为Fraction(4,1)，然后调用operator+
```

- Explicit-one-argument ctor

  explict关键字：表明不要让编译器自动将其他类型转化为类实例

```c++
class Fraction{
  public:
  	explict Fraction(int num,int den=1):m_numerator(num),m_denominator(den){}
  	Fraction operator+(const Fraction& f){
      return Fraction(...);
    }
  private:
  	int m_numerator;   //分子
  	int m_denominator;  //分母
}

Fraction f(3,5);
Fraction d2= f+4 ; //error:无法将double转换为Fraction
```

## 4.pointer-like classes

### 智能指针

```c++
template<typename T>
class shared_ptr{
public:
  T& operator*(){
    return *px;
  }
  T* operator->(){
		return px;
  }
  shared_ptr(T* p):px(p){}
  ...
private:
  T* px;
  long* pn;
}
```

### 迭代器

```c++
reference operator*() const{
  return (*node).data;
}
pointer operator->() const{
  return &(operator*());
}

list<Foo>::iterator ite;
...
*ite;
ite->method();
```

## 5.function-like classes 仿函数

```c++
template<class T>
  struct identity {
    const T& operator()(const T& x) const { return x;}
  };s
```



### 6.namespace



## 7.类模板

## 8.函数模板

## 9.成员模板

- 构造函数中经常用到
- 比如鱼类的类构造函数参数可以放入鲫鱼类对象

## 10.模板特化

```c++
template<class Key>
struct hash();

template<>
struct hash<char>{
  size_t operator()(char x) const {return x;}
}
```

## 11.模板偏特化

- 个数偏特化

```c++
teplate<typename T, typename Alloc=...>
class vector{
  ...
};

template<typename Alloc=...>
class vector<bool,Alloc>{
  ...
}
```

- 范围偏特化

```c++
template<typename T>
class C{
  
}


template<typename T>
class C<T*>{
  
}

C<string> obj1;
C<string*> obj2; 
```



## 12.模板模板参数

```c++
template<typename T,
	template<typename T>
				class SmartPtr
   >
class XCls{
  private:
  	SmartPtr<T> sp;
  public:
}


XCLs<string,shared_ptr> p1;
```

## 14 variadic templates && auto && for(dec1:coll)

```c++
void print(){
}
template<typename T,typename... Types>
void print(const T firstArg,const Types... args){
  cout << firstArg << endl;
  print(args...);
}
```

```c++
vector<double> vec;
...
for(auto elem:vec){
	cout << elem << endl;
}
for(auto& elem : vec){
	elem*=3;
}
```

## 15.reference

代表变量

虽然内部实现是指针，但会造成假象 大小与变量大小相等；实际底层只占指针大小

常用在参数和返回值类型



## 17.虚指针和虚函数表 <重要>

![image-20200313010352046](C++%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1.assets/image-20200313010352046.png)

