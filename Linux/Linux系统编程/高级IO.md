# 高级IO

B站《Linux系统编程、网络编程》第9章 高级IO

## 1.非阻塞IO

读普通文件不阻塞

读键盘，鼠标，管道等会阻塞



- 实现非阻塞读

  - 1.打开文件时，指定O_NONBLOCK

    ```c++
    open($filepath,O_RDONLY|O_NONBLOCK);
    ```

  - 2.通过fcntl函数指定O_NONBLOCK来实现

    当无法通过open函数设置非阻塞时，可以采用fcntl进行重设或补设

    ```c++
    //重设
    fcntl($fd,F_SETFL,O_RDONLY|O_NONBLOCK);
    
    //补设
    flag=fcntl(0,F_GETFL);//获取旧的文件状态标志
    flag=flag|O_NONBLOCK;
    fcntl($fd,F_SETFL,flag);
    ```

- 实现同时“读键盘”和“读鼠标”

  - 1.使用多进程
  - 2.使用多线程
  - 3.非阻塞循环读



## 2.记录锁(文件锁)

- 作用

  保护文件数据

  多进程共享读写同一个文件时，为了不让进程们各自读写数据时相互干扰，可以使用进程信号量来互斥实现，除此之外，也可以使用文件锁来实现，而且功能丰富，使用起来相对容易些

- 多进程读写文件

  - 写写互斥
  - 读写互斥
  - 读读共享

- 文件锁实现

  ```c++
  int fcntl(int fd,int cmd, .../*struct flock *flockptr */);
  
  struct flock{
    short l_type; //F_RDLCK,F_WRLCK,F_UNLCK
    short l_whence; //SEEK_SET,SEEK_CUR,SEEK_END
    off_t l_start; //相对于whence的精定位
    off_t l_len; //对多长的内容加锁,0表示加到文件末尾
    off_t l_pid; //加锁的进程pid
  }
  
  
  //示例代码
  
  //非阻塞设置写锁
  #define SET_WRFLCK(fd,l_whence,l_offset,l_len) set_filelock(fd,F_SETLK,F_WRLCK,l_whence,l_offset,l_len)
  //非阻塞设置读锁
  #define SET_WRFLCK(fd,l_whence,l_offset,l_len) set_filelock(fd,F_SETLK,F_RDLCK,l_whence,l_offset,l_len)
  //阻塞设置写锁
  #define SET_WRFLCK(fd,l_whence,l_offset,l_len) set_filelock(fd,F_SETLKW,F_WRLCK,l_whence,l_offset,l_len)
  //阻塞设置读锁
  #define SET_WRFLCK(fd,l_whence,l_offset,l_len) set_filelock(fd,F_SETLKW,F_RDLCK,l_whence,l_offset,l_len)
  //解锁
  #define SET_WRFLCK(fd,l_whence,l_offset,l_len) set_filelock(fd,F_SETLK,F_UNLCK,l_whence,l_offset,l_len)
  
  //该函数可以实现阻塞加读锁/阻塞加写锁，非阻塞加读锁/非阻塞加写锁/解锁
  void set_filelock(int fd,int ifwait,int l_type,int l_whence,int l_offset,int l_len){
    int ret = 0;
    struct flock flck;
    flck.t_type = l_type;
    flck.l_whence=l_whence;
    flck.l_start = l_offset;
    flck.l_len = l_len;
    ret = fcntl(fd,ifwait,&flck);
  }
  
  open(...);
  ret = fork();
  if(ret > 0){
    while(1){
      SET_WRFLCKW(fd,SEEK_SET,0,0);
      write(fd,"hello ",6);
      write(fd,"world\n",6);
      UNLCK(fd,SEEK_SET,0,0);
    }
  }else if(ret == 0){
    while(1){
      SET_WRFLCKW(fd,SEEK_SET,0,0);
      write(fd,"hello ",6);
      write(fd,"world\n",6);
      UNLCK(fd,SEEK_SET,0,0);
    }
  }
  ```

- 文件锁原理

  子进程继承父进程(fork之前open文件，fork之后打开文件是不共享文件表的) 

  ![image-20200410162411178](%E9%AB%98%E7%BA%A7IO.assets/image-20200410162411178.png)

  ![image-20200410162558415](%E9%AB%98%E7%BA%A7IO.assets/image-20200410162558415.png)

  

  

  文件锁：进程共享文件锁链表，通过检查链表上的锁节点，从而知道自己能不能操作；

  信号量：进程间共享信号量集合，通过检查集合中信号量的值，从而知道自己能不能操作

- 注意

  - 父进程加文件锁，子进程不会继承
  - 在同一进程，如果多个文件描述符指向同一文件，只要关闭其中任何一个文件描述符，该进程加在文件上的文件锁就会被删除，也就是该进程在文件锁链表上的读锁写锁节点会被删除
  - 进程终止时会关闭所有打开的文件描述符，所以进程结束时会自动删除所有加的文件锁
  - 多线程之间也可以使用fcntl实现的文件锁，但是线程不能使用同一个open返回的文件描述符，线程必须使用自己open所得到的文件描述符才有效

- 使用flock函数实现文件锁

  - flock与fcntl锁实现的文件锁一样，可以用到多进程和多线程

  - ```c++
    int flock(int fd,int operation);
    //operation
    // LOCK_SH 共享锁
    // LOCK_EX 互斥锁
    // LOCK_UN 解锁
    
    ```

  - 注意

    - flock用于多进程时，各进程必须独立open打开文件，父子进程也必须各自独立open打开文件；
      - 用于多线程一样，也必须在各线程都独立open打开文件
    - 而在fcntl中，父子进程可以使用继承下来的文件描述符来加锁；多线程需要独立open打开获得文件描述符，来加锁

## 3.IO多路复用

- 读键盘读鼠标

  - 多进程/多线程
  - 非阻塞
  - 多路IO复用
  - 异步IO

- 代码演示

  ```c++
  select/poll
  ```

## 4.异步IO

使用异步IO的前提是，底层驱动必须有发送SIGIO信号的代码，比如鼠标和键盘底层驱动就实现了发送SIGIO信号的代码

步骤：

- 调用signal函数对SIGIO信号设置捕获函数，在捕获函数里面实现读操作，比如读鼠标

  ```c++
  signal(SIGIO,signal_fun);
  ```

- 使用fcntl函数，将接收SIGIO信号的进程设置为当前进程

  ```c++
  fcntl(mousefd,F_SETOWN,getpid);
  ```

- 使用fcntl函数，对文件描述符增设O_ASYNC的状态标志，让fd支持异步IO

  ```c++
  flag=fcntl(mousefd,F_GETFL);//获取旧的文件状态标志
  flag=flag|O_ASYNC;
  fcntl(mousefd,F_SETFL,flag);
  ```



## 5.存储映射

普通文件读写read/write会消耗很多时间，会从应用层调用到系统层，从应用层缓存倒腾到内核缓存，最后写入文件。

当涉及大数据量时，read/write的中间过程复杂，效率很低，因此需要用到内存映射



- mmap函数
  - 通常情况下，进程空间的虚拟地址只映射自己底层物理空间的物理地址，但是使用mmap时，他会将文件的硬盘空间地址也映射到虚拟内存空间，这么一来应用程序就可以直接通过映射的虚拟地址操作文件，根本就不需要read，write函数了，使用地址操作时省去了繁杂的中间调用过程，可以快速对文件进行大量数据的输入输出
  - 具体映射到：进程应用空间堆栈之间的区域



- 存储映射与共享内存

  - 存储映射：其实也可以实现进程间通信。但映射到硬盘，效率自然比共享内存低

    ​	比如AB进程都映射到同一个普通文件里

  - 用途不同

    - 共享内存实现进程间通信
    - 存储映射实现大量数据的高效输入输出

- mmap函数实现

  ```c++
  //addr：人为指定映射的起始虚拟地址，如果设置为NULL，表示由内核决定
  //length：映射长度 
  //prot：指定映射区域的操作权限，PROT_EXEC,PROT_WRITE,PROT_READ,PROT_NONE,读写执行不允许操作
  //flags:向映射区域写入了数据，是否将数据立即更新文件中
  //fd:映射文件的描述符
  //文件头offset的位置开始映射
  void* mmap(void* addr,size_t length,int prot,int flags,int fd,off_t offset);
  
  //取消映射
  munmap(void* addr,size_t length);
  ```

- 代码演示

  ```c++
  //将文件A大量数据写入B文件中
  ```

  mmap如果映射的file为空文件(长度为0)时，内核会像进程发送SIGBUS信号，会异常退出

  需要使用truncate函数截断B文件长度为A文件长度

