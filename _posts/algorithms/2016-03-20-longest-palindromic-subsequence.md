---
title: 最长回文子序列 Longest Palindromic Subsequence
category : algorithms
tags: 字符串 DP 回文串
---

### # 题意
Given a sequence, find the length of the longest palindromic subsequence in it. For example, if the given sequence is “BBABCBCAB”, then the output should be 7 as “BABCBAB” is the longest palindromic subseuqnce in it. “BBBBB” and “BBCBB” are also palindromic subsequences of the given sequence, but not the longest ones.

### # 思路
设目标字符串为str。对于长度为1的字串，因为一个字符也是回文的，所以LPS(str[i,i])为1。对于长度大于1的字串，设从位置i开始的长度为k+1(k>=1)的字符串，str[i, i+k]有：

1.若str[i]与str[i+k]不相等，则LPS(str[i, i+k])等于该区间内长度为k的字串的LPS。即：

    LPS(str[i, i+k]) = max{LPS(str[i, i+k-1]), LPS(str[i+1, i+k])}

2.若str[i]与str[i+k]相等，则

    LPS(str[i, i+k]) = LPS(str[i+1, i+k-1]) + 2

有如下性质，所以当str[i]等于str[i+k]时，LPS直接计算即可。

    LPS(str[i+1, i+k-1]) + 2 >= max{LPS(str[i, i+k-1]), LPS(str[i+1, i+k])}

因此采用DP的方式计算。代码如下

```c++
int LPS(const string &str){
    int n = str.length();
    int dp[n][n];

    memset(dp, 0, sizeof(dp));
    for(int i=0;i<n;++i) dp[i][i]=1;

    for(int k=1;k<n;++k){
        for(int i=0;i+k<n;++i){
            int j = i+k;

            if(str[i] == str[j]){
                dp[i][j] = dp[i+1][j-1]+2;
            }
            else
                dp[i][j] = max(dp[i][j-1], dp[i+1][j]);
        }
    }
    return dp[0][n-1];
}
```
