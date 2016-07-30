---
layout: post
title: Algorithm Problems: DP
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

### 10 - Regular Expression Matching

**1. DP**

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        bool dp[p.length()+1][s.length()+1];

        dp[0][0] = true;

        for(int j=1;j<=s.length();++j)
            dp[0][j] = false;

        for(int i=1;i<=p.length();++i)
            dp[i][0] = i>=2 && p[i-1]=='*' && dp[i-2][0];

        for(int i=1;i<=p.length();++i) {
            for(int j=1;j<=s.length();++j) {
                if(p[i-1] == '*' && i>=2) {
                    dp[i][j] = dp[i-2][j] || ((p[i-2]==s[j-1] || p[i-2]=='.') && dp[i][j-1]);
                }
                else dp[i][j] = dp[i-1][j-1] && (p[i-1] == s[j-1] || p[i-1] == '.');
            }
        }

        return dp[p.length()][s.length()];
    }
};
```

**2. 回溯**

```c++
class Solution {
public:
    bool isMatch(string s, string p) {
        return foo(s, 0, p, 0);
    }

    bool foo(string s, int i, string p, int j) {
        if(i==s.length() && j==p.length())
            return true;

        if(j<p.length()-1 && p[j+1]=='*') {
            if(foo(s, i, p, j+2)) return true;

            for(int k=i;k<s.length()&&(s[k]==p[j] || p[j]=='.');++k) {
                if(foo(s, k+1, p, j+2)) return true;
            }

            return false;
        }
        else if(s[i]==p[j] || p[j]=='.')
            return foo(s, i+1, p, j+1);
        else return false;
    }
};
```
