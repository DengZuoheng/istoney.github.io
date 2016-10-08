---
layout: post
title: Google APAC Problem Solutions
category: Job Hunting
tags: [job]
---
{% include JB/setup %}

## Practice Round

### Problem A. Lazy Spelling Bee

> 给定单词W，按照以下规则生成可接受的答案A，A的第i个字符是W的第i-1、i或i+1个字符。0<=i-1,i,i+1<=W.length。

对A的i位置，检查可能的字符的个数，若W的i-1, i, i+1三个字符都相等，则只有一个可能性，有一对相等则有两个可能性，都不相等则有三个可能性。将结果累乘得最终结果。

```c++
#include<iostream>
#include<fstream>
using namespace std;

int main() {
    ifstream in("A-large.in");
    ofstream out("A-large.out");

    int T, t = 1;
    in>>T;

    while(t <= T) {
        string str;
        in>>str;

        long long ans = 1;

        if(str.length() > 1 && str[0] != str[1])
            ans *= 2;

        for(int i=1;i<str.length()-1;++i) {
            int count = 1;
            if(str[i-1] == str[i] && str[i] == str[i+1])
                count = 1;
            else if(str[i-1] == str[i] || str[i-1] == str[i+1] || str[i] == str[i+1])
                count = 2;
            else count = 3;
            ans = ans*count % 1000000007;
        }

        if(str.length() > 1 && str[str.length()-2] != str[str.length()-1])
            ans = ans*2 % 1000000007;

        out<<"Case #"<<t<<": "<<ans<<endl;
        ++t;
    }

    in.close();
    out.close();

    return 0;
}
```

### Problem B. Robot Rock Band

> 机器人乐队共有四个位置，每个位置上有N的机器人可供选择，每个机器人由一个整数数字表示，机器人的数字可能相同。当四个位置上机器人的数字亦或（XOR）结果为K时，可获得最好的表演效果。

\\( 0 \leq N \leq 1000 \\)

\\( 0 \leq K \leq 10^9 \\)

\\( 0 \leq all robots numbers \leq 10^9 \\)

每个位置有1000种可能性，全部遍历则有10^9中可能，不可行。可以先计算前两个位置的亦或结果，存在哈希表中，然后遍历后两个位置，然后从哈希表查找对应结果的个数。

```c++
#include<stdio.h>
#include<stdlib.h>
#include<iostream>
#include<string>
#include<fstream>
#include<map>
using namespace std;

int main() {
    ifstream in("B-large.in");
    ofstream out("B-large.out");

    int A[4][1000];
    int T, N, K, tmp;
    in>>T;

    for(int t=1;t<=T;++t) {
        cout<<t<<endl;
        in>>N>>K;

        map<int, int> M;

        for(int i=0;i<4;++i) {
            for(int j=0;j<N;++j) {
                in>>A[i][j];
            }
        }

        for(int i=0;i<N;++i) {
            for(int j=0;j<N;++j) {
                M[A[0][i]^A[1][j]]++;
            }
        }

        long long ans = 0;
        for(int i=0;i<N;++i) {
            for(int j=0;j<N;++j) {
                tmp = K^A[2][i]^A[3][j];
                ans += M[tmp];
            }
        }

        out<<"Case #"<<t<<": "<<ans<<endl;
    }

    in.close();
    out.close();

    return 0;
}
```
