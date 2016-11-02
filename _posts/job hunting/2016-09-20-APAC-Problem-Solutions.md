---
layout: post
title: 2016 Google APAC Problem Solutions
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

### Problem C. Not So Random

> 现有随机数生成器RNG，以非负整数作为输入，给定整数K，做如下操作：
- 以A/100的概率，将输入与K相与；
- 以B/100的概率，将输入与K相或；
- 以C/100的概率，将输入与K亦或；
现有N个RNG，将其串联，求输入X的输出期望。

\\( 1 \leq N \leq 10^5 \\)

\\( 0 \leq X \leq 10^9 \\)

\\( 0 \leq K \leq 10^9 \\)

```c++
#include<stdio.h>
#include<stdlib.h>
#include<iostream>
#include <iomanip>
#include<string>
#include<fstream>
#include<map>
using namespace std;

double foo(int N, int X, int K, int A, int B, int C) {
    if(N == 0) return 0;
    double a = (double)A/100, b = (double)B/100, c = (double)C/100;
    map<int, double> table;

    table[X] = 1;
    for(int i=0;i<N;++i) {
        map<int, double> next;
        for(auto it=table.begin();it!=table.end();++it) {
            int val = it->first;
            double p = it->second;
            next[val & K] += p*a;
            next[val | K] += p*b;
            next[val ^ K] += p*c;
        }
        table = next;
    }

    double ans = 0.0;
    for(auto it=table.begin();it!=table.end();++it) {
        int val = it->first;
        double p = it->second;
        ans += val*p;
    }
    return ans;
}

int main() {
    ifstream in("C-large-practice.in");
    ofstream out("C-large-practice.out");
    out<<setiosflags(ios::fixed)<<setprecision(10);

    int T, N, X, K, A, B, C;
    in>>T;
    for(int t=1;t<=T;++t) {
        in>>N>>X>>K>>A>>B>>C;
        cout<<t<<endl;
        out<<"Case #"<<t<<": "<<foo(N, X, K, A, B, C)<<endl;
    }
    in.close();
    out.close();
    return 0;    
}
```
