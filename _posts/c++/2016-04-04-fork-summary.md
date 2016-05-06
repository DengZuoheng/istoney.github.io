---
layout: post
title: Unix 多进程 fork 函数总结
category : c++
tags: [fork, 多进程]
---
{% include JB/setup %}

### # 进程
操作系统中，进程是具有一定功能的程序在某个数据集合上的一次运行活动，是进行资源分配和调度的最小单位(在引入线程的操作系统中，线程是最小的调度单位)。一个进程中，主要包含三个元素：

 - 可执行程序
 - 与该进程相关的全部数据(包括变量、内存空间、缓存区等)
 - 程序执行的上下文

### # fork函数
fork()是Unix的系统调用函数，用来创建一个新的进程。

```c++
#include<unistd.h>

// On success, The PID of the process is returned in the parent, and 0 is returned in the child.
// On failure, -1 is returned in the parent, no child process is created, and errno is set appropriately.
pid_t fork(void);
```
调用fork()成功后，将生成一个新的进程，即子进程。子进程拥有自己独立的堆栈、数据段等，其数据是父进程的完全拷贝。但从性能考虑，会采用copy-on-write。除了少数值不同外，子进程相当于是父进程的克隆。

子进程创建成功后，将从fork()的返回值开始继续执行。但是父进程和子进程的执行顺序无法确定。

fork()调用的返回值为：

1. 调用成功，在父进程中返回子进程id;
2. 调用成功，在子进程中返回0；
3. 调用失败，在父进程中返回-1；

同时Unix提供系统调用来查看进程id，getpid()函数可以获取当前进程的id，getppid()则可以获取当前进程的父进程id。在父进程调用这两个函数的结果是相同的。

### # fork实例分析
有如下fork()调用的c++代码，求最终输出K的数目。

```c++
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>

int main(){
    for(int i=0;i<2;++i){
        fork();
        printf("K");
    }
    wait(NULL);
    return 0;
}
```
对上述程序分析，可得：

 - i=0时，调用fork()一次，得到两个进程①和②，分别输出一次K;
 - i=1时，进程①、②分别调用fork()，生成新进程③和④，四个进程分别输出一次K;

其结构如下图所示：

![进程结构](/assets/post_img/c++/fork-processes.jpg)

则共输出去6个K。但是答案并不是6，而是8，如下图。

![fork()示例输出结果](/assets/post_img/c++/fork-output-0.jpg)

#### # 输出进程id
为了验证程序执行过程中子进程的生成情况，在输出K的时候，将i的值和当前进程的id同时输出。

```c++
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>

int main(){
    printf("parent pid: %d\n",getpid());

    for(int i=0;i<2;++i){
        fork();
        //printf("K");
        printf("K-%d-%d ", i, getpid());
    }
    wait(NULL);
    return 0;
}
```
其结果如下图：

![输出i和pid](/assets/post_img/c++/fork-output-1.jpg)
可见程序执行中确实只有四个进程，但是进程①和②却分别多输出了一次。

#### # 加上时间戳
为了进一步分析其原因，在每次输出的时候加上时间戳，用以判断多输出的两次是何时输出的。

采用Unix的time库进行高精度的计时。

```c++
#include <sys/time.h>
timeval t;
gettimeofday(&t,0);

//timeval的结构如下：
strut timeval{
    long tv_sec; //秒数
    long tv_usec; //微秒数
};
```
示例程序如下：

```c++
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<sys/time.h>

int main(){
    printf("parent pid: %d\n",getpid());
    timeval t;

    for(int i=0;i<2;++i){
        fork();
        gettimeofday(&t, 0);
        //printf("K");
        //printf("K-%d-%d ", i, getpid());
        printf("K-%d-%d-%ld ", i, getpid(), t.tv_usec);
    }
    wait(NULL);
    return 0;
}
```
运行结果如图

![输出i和pid](/assets/post_img/c++/fork-output-2.jpg)

可见进程①和进程②的第一次输出，被输出了两次，联想到c++的标准输出具有缓存机制，有可能第一次输出时，将输出内存存放在缓存中。在执行fork()的时候，io缓存也被拷贝到子进程中，因此重复输出。

但是程序中printf()是在fork()之后执行的，不清楚printf()的输出缓存是否是在fork()之前执行。

#### # 刷新输出缓存
在每次输出后换行，刷新输出缓存。

```c++
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>
#include<sys/time.h>

int main(){
    printf("parent pid: %d\n",getpid());
    timeval t;

    for(int i=0;i<2;++i){
        fork();
        gettimeofday(&t, 0);
        //printf("K");
        //printf("K-%d-%d ", i, getpid());
        //printf("K-%d-%d-%ld ", i, getpid(), t.tv_usec);
        printf("K-%d-%d-%ld\n", i, getpid(), t.tv_usec);
    }
    wait(NULL);
    return 0;
}
```
运行结果如下，因此多输出的两次就是因为输出缓存也被子进程拷贝，造成重复输出。

![fork()示例输出结果](/assets/post_img/c++/fork-output-3.jpg)
