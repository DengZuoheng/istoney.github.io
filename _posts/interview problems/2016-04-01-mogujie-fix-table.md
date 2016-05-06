---
layout: post
title: 蘑菇街2016笔试题 修理桌子
category : interview problems
tags: [递归]
---
{% include JB/setup %}

### # 题意
Arthur买了一张大桌子，但是桌子腿的长度不相同，导致桌子不稳，需要修理。桌子有n条腿，第i条腿长度为\\(l_i\\)，移除第i条腿的代价为\\(d_i\\)。桌子平稳的条件是超过一半的桌子腿长度达到最大长度。计算使桌子平稳的最小代价。

**输入**<br>
第一行是一个整数：n (\\(1\le n\le 10^{5}\\)), 表示桌腿总数。
第二行是n个整数：\\(l_1\\), \\(l_2\\), ..., \\(l_n\\) (\\(1\le l_i\le10^{5}\\)), 表示每条桌腿的长度。
第三行是n个整数：\\(d_1\\), \\(d_2\\), ..., \\(d_n\\) (\\(1\le d_i\le200\\)), 表示移除每条桌腿的代价。

**输出**<br>
一个整数，表示使桌子变平稳的最小代价

**样例输入**<br>

    6
    2 2 1 1 3 3
    4 3 5 5 2 1

**样例输出**<br>

    8

### # 思路
考虑目前最长的桌腿，设其长度为lgst，数量为count，cost总和为total。针对lgst有两种可能的情况：

- 保留长度为lgst的桌腿，此时需要使count大于桌腿数的一半；若count>n/2则不需要移除，否则需要从剩余桌腿中移除k=n-2*count+1条；选取cost最小的k个即可。
- 移除长度为lgst的桌腿，需要将长度为lgst的桌腿，全部移除，然后从剩余桌腿的寻找稳定方案，进入递归。

### # 源码
```c++
#include<stdio.h>
#include<iostream>
#include<queue>
#include<map>
using namespace std;

typedef pair<int, int> P;
P legs[100005];

struct leg{
    int count = 0;
    int total = 0;
    vector<int> costs;
};
map<int, leg> table;

int f(map<int, leg>::iterator it, int n, int cost){
    leg lgst = it->second;

    if(lgst.count > n/2) return cost;

    int res1 = f(--it, n-lgst.count, cost+lgst.total);

    ++it;
    int res2 = cost;
    int k = n - 2*lgst.count + 1;
    priority_queue<int, vector<int>, greater<int>> Q;

    for(map<int, leg>::iterator i = table.begin();i!=it;++i){
        leg tmp = i->second;
        for(int j=0;j<tmp.costs.size();++j){
            if(Q.size() < k) Q.push(tmp.costs[j]);
            else{
                if(tmp.costs[j] < Q.top()){
                    Q.pop();
                    Q.push(tmp.costs[j]);
                }
            }
        }
    }

    while(!Q.empty()){
        res2 += Q.top();
        Q.pop();
    }

    return min(res1, res2);
}

int main(){
    int n, tmp, l;
    scanf("%d", &n);
    for(int i=0;i<n;++i) scanf("%d", &legs[i].first);
    for(int i=0;i<n;++i) scanf("%d", &legs[i].second);

    for(int i=0;i<n;++i){
        l = legs[i].first;
        table[l].count++;
        table[l].total += legs[i].second;
        table[l].costs.push_back(legs[i].second);
	}

    int res = f(--table.end(), n, 0);
    printf("%d\n", res);

    return 0;
}
```
