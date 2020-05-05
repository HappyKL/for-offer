# 文件IO

## 1.常用函数

```c++
open
read
write
lseek
close
```

 ## 2.open流程

- 记录打开文件的消息
  - 程序运行起来后就是一个进程了，os会创建一个task_struct的结构体，记录进程运行时的各种信息，比如所打开文件的相关信息
  - open将文件成功打开后，在task_struct中又会创建一些结构体，用于记录当前进程目前所打开文件的信息，后续所有的文件操作，都需要依赖于这些信息，其中就包括指向打开文件的文件描述符
- open函数会申请一段内存空间(内核缓存)，后续读写文件时，用于临时缓存读写文件时的数据
  - 读写数据倒腾流程： write->应用缓存->内核缓存->驱动缓存->文件 

## 3.open函数1

```c++
O_RDONLY
O_WDWR
O_WRONLY

```



## 4.open函数2与文件描述符

```c++
O_CREAT
O_EXCL //保证每次open打开的是新文件，如果文件已存在，则报错
```

- 文件描述符
  - 系统会给进程分配0～1023文件描述符范围
  - 1023可以改，但没这个必要
  - open打开的是最小的没用的文件描述符

## 5.errno和函数错误号

当函数出错时，系统会给errno赋予错误号

```c++
perror("open file");
// open file : file not exist
```

## 6.close,read,write

- close

  ```c++
  close(fd);//不主动关闭，当进程结束时，会自动关闭，但有时因为某种需要，还是需要自己手动关闭的
  ```

  - close关闭文件时发生了啥
    - open打开文件时，会在进程的task_struct结构中，创建相应的结构体，以存放打开文件的相关信息
    - close时，需要释放存放文件打开时信息的结构体，类似于malloc和free，只不过malloc和free是C应用程序调用的库函数，Linux系统内部开辟和释放空间时，用的是自己特有的函数

- write

  ```c++
  write(int fd,const void* buf ,size_t count);
  //向fd所指向的文件写入数据
  ```

  数据中转过程：应用缓存(buf)->open打开文件时开辟的内核缓存->驱动程序的缓存->块设备的文件 

- read

  与write类似

## 7.标准输入文件

/dev/stdin 键盘

scanf底层调用read(0,buf,**);

## 8.标准输出和标准错误输出

/dev/stdout 显示器

printf底层调用write(1,buf,size(buf));



/dev/stderr 错误输出到显示器

perror底层调用write(2,buf,..)

## 9.进程表与文件描述符表《重要》

- 文件描述符表：当open打开文件成功后，会创建相应的结构体，用于保存被打开文件的相关信息，对文件进行读写等操作等，会用到这些信息，这个数据结构就是“文件描述符表”
- 进程表 task_struct
  - 成员多达近300个
  - 每一个进程运行起来都会有一个task_struct存在
  - 文件描述符表被包含在task_struct
  - 进程结束后，进程表会被释放
- ![image-20200410111201114](%E7%AC%AC1%E7%AB%A0-%E6%96%87%E4%BB%B6IO.assets/image-20200410111201114.png)
  - 函数指针：read,write等操作文件时，会根据底层具体情况的不同，调用不同的函数来实现读写，所以在V节点里面保存了不同函数的函数指针
  - i节点信息：存储文件属性

## 10.文件共享操作1

- 同一进程open两次文件

![image-20200410120626712](%E7%AC%AC1%E7%AB%A0-%E6%96%87%E4%BB%B6IO.assets/image-20200410120626712.png)

- 解决相互覆盖：可以设置open状态O_APPEND,从而避免多次操作文件相互覆盖的情况

  ```c++
  fd1 = open(FILE_NAME,O_RDWR|O_APPEND);
  fd2 = open(FILE_NAME,O_RDWR|O_APPEND);
  ```

## 11.文件共享操作2

- 多个进程打开同一个文件

![image-20200410121858666](%E7%AC%AC1%E7%AB%A0-%E6%96%87%E4%BB%B6IO.assets/image-20200410121858666.png)

- 解决相互覆盖：同样可以使用O_APPEND

## 12.dup和dup2

- dup

  ```c++
  //复制某个已经打开的文件描述符，新描述符也指向相同的文件
  int dup(int oldfd);
  
  //演示
  int main(){
    int fd1 = 0;
  	int fd2 = 0;
  	fd1 = open(FILE_NAME,O_RDWR|O_TRUNC);
  	fd2 = dup(fd1);
  
  	printf("fd1 = %d,fd2 = %d\n",fd1,fd2); //3,4
  
  	write(fd1,"hello\n",6);
  	write(fd2,"world\n",6);
    return 0;
  }
  ```

- dup2

  ```c++
  //可以指定新文件描述符，如果新描述符已打开，则关闭后重新打开，并与oldfd指向相同文件
  int dup2(int oldfd,int newfd);
  
  //演示
  int main(){
    int fd1 = 0;
  	int fd2 = 0;
  	fd1 = open(FILE_NAME,O_RDWR|O_TRUNC);
  	fd2 = dup2(fd1,4);
  
  	printf("fd1 = %d,fd2 = %d\n",fd1,fd2); //3,4
  
  	write(fd1,"hello\n",6);
  	write(fd2,"world\n",6);
    return 0;
  }
  ```

- 原理

  ![image-20200410164625077](%E7%AC%AC1%E7%AB%A0-%E6%96%87%E4%BB%B6IO.assets/image-20200410164625077.png)

  多个文件描述符，只有一个文件表，共享文件位移量，因此不会相互覆盖

## 13.dup、dup2实现重定位

- 重定位

  某描述符原来指向A文件，输出数据输出到A文件，但是重定位后，文件描述符指向B文件，使得输出的目标文件也随之改变

- 实现步骤

  ```c++
  //newfd = open file
  //close(oldfd)
  //使用dup,dup2把oldfd指向newfd指向的文件
  
  //举例:将printf输出到文件而不是屏幕
  fd1 = open(FILE_NAME,O_RDWR|O_TRUNC);
  close(1);
  dup(fd1); //或者dup2(fd1,1);
  printf("hello\n"); //从而输出到file中
  
  ```

- 使用场景

  函数中文件描述符写死，无法修改，而又希望把数据输出到其他文件时

- 应用实例

  ```shell
  ls > file.txt # 底层实现其实就是上面举的例子dup2(fd,1);
  ```

- 总结文件共享

  - 同一进程多次open
  - 多进程open
  - 单进程dup/dup2



## 14.fcntl函数

```c++
fcntl(int fd,int cmd,.../*arg */);
//fd:文件描述符
//cmd:控制命令，通过指定不同的宏来修改fd所指向文件的性质
	//F_DUPFD 模拟dup和dup2
	//F_GETFL,F_SETFL
  //F_GETFD，F_SETFD
  //F_GETOWN,F_SETOWN
  //F_GETLK,F_SETFK,F_SETLKW
```

