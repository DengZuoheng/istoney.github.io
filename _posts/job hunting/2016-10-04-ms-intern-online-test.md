---
layout: post
title: MS 2016 Intern Online Test
category: Job Hunting
tags: [job]
---
{% include JB/setup %}

## Problem A. Font Size

> 在宽为W，高为H的手机屏幕上显示N段文字，每段文字字符个数为\\( a_i \\)。当字符size为S*S时，每行显示字符数为⌊W / S⌋，每页显示⌊H / S⌋行。段落之间没有空行。求总页数不超过P时，最大字符size。

字符最小size为1，最大为min(W, H)，二分搜索最优字符大小。二分搜索结束条件为：字符大小S对应页数为P；S对应页数小于P，S+1对应页数大于P。

```c++
#include<stdio.h>
#include<iostream>
using namespace std;

int paras[1005];

int calPages(int N, int W, int H, int S) {
    int pages = 0;
    int lines = 0;
    int chsInLine = W/S;
    int linesInPage = H/S;

    for(int n=0;n<N;++n) {
        lines += paras[n]/chsInLine;
        lines += (paras[n]%chsInLine) ? 1 : 0;

        pages += lines/linesInPage;
        lines = lines%linesInPage;
    }

    pages += lines ? 1 : 0;

    return pages;
}


int main() {
    int T, N, P, W, H;
    scanf("%d", &T);

    for(int t=1;t<=T;++t) {
        scanf("%d%d%d%d", &N, &P, &W, &H);

        for(int i=0;i<N;++i) {
            scanf("%d", &paras[i]);
        }

        int s = 0, S = min(W, H);

        while(s < S) {
            int mid = (s + S) >> 1;
            int p = calPages(N, W, H, mid);
            if(p == P) {
                s = mid;
                break;
            }
            if(p > P) {
                S = mid - 1;
                continue;
            }

            int p2 = calPages(N, W, H, mid+1);
            if(p2 == P) {
                s = mid + 1;
                break;
            }
            if(p2 > P) {
                s = mid;
                break;
            }

            s = mid + 1;
        }

        printf("%d\n", s);
    }
    return 0;
}
```
