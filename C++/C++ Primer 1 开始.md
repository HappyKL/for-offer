# 1.1 编写程序

## 1.1.1 编译运行程序

```shell
# -std=c++0x该参数可打开对C++11的支持
g++ -o test test.cpp
./test
```

# 1.2 初识输入输出

```c++
//iostream库  基本类型istream和ostream
#include<iostream>
//标准库定义了4个IO对象
istream cin   //输入
ostream cout  //输出
ostream cerr   //错误
ostream clog   //一般性信息
    
std::cout << "hello world" << std::endl;
//<<云算法=符接收两个运算对象，左侧运算对象必须是一个ostream对象，右侧运算对象是一个要打印的值，该运算符将给定的值写入到给定的ostream对象中，输出结果就是其左侧运算对象；>>同理

//endl是一个被称为“操纵符”的特殊值，写入endl的效果是结束当前行，并将与设备关联的缓冲区中的内容刷到设备中

```



# 1.3 注释

`//          /**/`

# 1.4 控制流

## 1.4.1 while语句

## 1.4.2 for语句

## 1.4.3 读取不定的输入数据

```c++
int value
while(std::cin>>value)  //当使用istream对象作为条件时，其效果是检测流的状态。如果流是有效的，即流未遇到错误，那么检测成功。当遇到文件结束符(end-of-file)或遇到一个无效输入时(例如读入的值不是一个整数),istream对象的状态会变为无效，处于无效状态的istream对象会使条件变假

//ctrl+D linux中的文件结束符
//ctrl+Z,之后按Enter windows中的文件结束符
 
```

## 1.4.4 if语句

# 1.5 类简介













