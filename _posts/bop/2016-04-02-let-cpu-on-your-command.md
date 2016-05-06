---
title: 编程之美 1.1 让CPU占用率曲线听你指挥
category: beauty of programming
tags: CPU
---

### # 题意
写一个程序，让用户决定Windows任务管理器的CPU占用率。实现如下三种情况：

1. CPU的占用率固定在50%，为一条直线；
2. CPU的占用率为一条直线，具体占用率由命令行参数决定(参数范围1~100)；
3. CPU的占用率是一条正弦曲线。

### # 思路
CPU占用率是指在统计时间内CPU处于busy状态的平均时间比。任务管理器的刷新时间是1s，所以只要让CPU在1s内交替处于busy和idle状态即可。要让CPU处于busy状态，只要使其持续执行指令，例如for循环即可。而让CPU处于空闲状态的方法有，处理IO、等待网络、调用Sleep函数。

#### # 解法1
让CPU在1s的时间内，一半的时间执行for循环，另一半的时间Sleep。但是如果选0.5s作为时间间隔的话，容易造成占用率波动，因此选择10ms(接近Windows CPU调度时间片)作为切换的时间间隔。

使CPU空闲的话，只要Sleep(10)即可。但是要使CPU busy 10ms需要for循环多少次呢？

首先可以查看CPU的处理频率，当前实验环境CPU为i5-3470 3.2GHz(四核，四线程，占用率针对单核计算，为0.5/4=0.125)。即，cpu时钟频率为SPEED=3.2*10^9。但通常执行一条指令需要几个时钟周期，假设t=2。则cpu 1s内执行的指令数为SPEED/t.

然后需要查看一下一个for循环共包含多少机器指令，通过对以下代码反编译，得到其汇编指令。

```c++
#include<stdio.h>
int main(){
    int i;
    for(i=0;i<100;++i) {}
    return 0;
}
```
![for循环汇编指令](/attaches/bop/1-1-forloop-assemble.jpg)

可以看到空的for循环，执行一次有三条汇编指令。因此设占用率为rate，idle时间是10ms，则for循环的循环次数为，(idletime/(1-rate) - idletime)\*SPEED/(3*t)。得到如下代码。

```c++
#include<stdio.h>
#include<stdlib.h>
#include<Windows.h>

using namespace std;

void keepCPUBusy(float rate){
    int i;
    if(abs(1.0-rate) < 0.0000001){
        while(true) ++i;
    }

    const int idletime = 10;
    const int SPEED = 3200000;
    const int t = 2;
    int count = (idletime/(1-rate) - idletime)*SPEED/(3*t);
    printf("%d\n", count);

    while(true){
        for(i=0;i<count;++i) {}
        Sleep(idletime);
    }
}

int main(){
    keepCPUBusy(0.5);
    return 0;
}
```
执行该程序，得到CPU为占用率在12%和13%之间切换(因为任务管理器的CPU占用率为整数)，长时间的平均占用率为12.60左右。

#### # 解法2
GetTickCount()方法可以获取“系统启动到现在”所经历时间的毫秒值，最多可以记录49.7天，因此可以利用GetTickCount()控制busy loop要循环多久。

```c++
void keepCPUBusy2(float rate){
	int i;
	if(1.0-rate < 0.0000001){
		while(true) ++i;
	}

	const int idletime = 10;
	const int busytime = idletime/(1-rate)-idletime;
	printf("%d\n", busytime);

	long long start = 0;
	while(true){
		start = GetTickCount();
		while((GetTickCount()-start) <= busytime) {}
		Sleep(idletime+7);    //额外增加7ms idle时间抵消while和start
	}
}
```
但是while循环和start取值会有时间消耗，所以需要在Sleep()中额外添加一些时间。

#### # 正弦曲线
首先选择一个busy和idle切换的周期，1s的话会造成占用曲线波动，10ms太短又会使占用率精度降低，因此选择slotTime=100(ms)，占用率精度为0.01.

并对一个正弦周期抽取180个样本(这样正弦周期为180*100ms=18s)，作为busyTime。在第i个切换周期内先执行busyTime[i]的循环，然后使cpu idel，时间为slotTime-busyTime[i]。

同时，线程在多个cpu之间切换时，也会因为切换开销和多个cpu核的频率不同而造成占用率不准确，因此采用SetThreadAffinityMask选择cpu执行该线程，避免线程迁移。

```c++
void keepCPUBusy3(){
	double PI = 3.141592653;
	int slotNum = 180;
	int slotTime = 100;

	int busyTime[slotNum];
	for(int i=0;i<slotNum;++i){
		busyTime[i] = (sin(PI*i*2/slotNum)+1)*slotTime/2;
	}

	SetThreadAffinityMask(GetCurrentProcess(),0x00000001);
	int start = 0;
	for(int i=0;;i=(i+1)%slotNum){
		start = GetTickCount();
		while(GetTickCount()-start <= busyTime[i]) {}

		Sleep(slotTime - busyTime[i]);
	}
}
```
