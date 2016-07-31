---
layout: post
title: LeetCode Problems - DP
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

### 44 - Wildcard Matching

**1. 回溯**

```c++
class Solution {
public:
    bool isMatch(const string& s, const string& p) {
        int i = 0, j = 0;
        stack<pair<int, int>> S;

        //string 末尾为'\0'
        while(i<=s.length() && j<=p.length()) {
            if(i==s.length() && j==p.length()) return true;


            if(p[j] == '*') {
                S.push(make_pair(i+1, j++));
                continue;
            }
            else if(s[i]==p[j] || p[j]=='?') {
                ++i;++j;
                continue;
            }

            if(!S.empty()) {
                pair<int, int> P = S.top();
                S.pop();
                i = P.first;
                j = P.second;
            }
            else return false;
        }

        return false;
    }
};
```

**2. DP**

```c++
class Solution {
public:
    bool isMatch(const string& s, const string& p) {
        bool dp[2][s.size()+1];
        dp[0][0] = true;

        for(int j=1;j<=s.length();++j) dp[0][j] = false;

        for(int i=1;i<=p.length();++i) {
            int I = i%2;
            int I_1 = (i-1)%2;

            dp[I][0] = dp[I_1][0] && p[i-1]=='*';

            for(int j=1;j<=s.length();++j) {
                if(p[i-1]=='*')
                    dp[I][j] = dp[I][j-1] || dp[I_1][j] || dp[I_1][j-1];
                else dp[I][j] = dp[I_1][j-1] && (p[i-1]==s[j-1] || p[i-1]=='?');

                //cout<<i<<" "<<j<<" "<<dp[i][j]<<endl;
            }
        }

        return dp[p.length()%2][s.length()];
    }
};
```

### 91 - Decode Ways

**DP**

```c++
class Solution {
public:
    int numDecodings(string s) {
        if(s.empty() || s[0]=='0') return 0;

        int dp[s.length()+1];

        dp[0] = 1;
        dp[1] = 1;

        for(int i=2;i<=s.length();++i) {
            if(s[i-1] == '0') {
                if(s[i-2]=='1' || s[i-2]=='2') dp[i] = dp[i-2];
                else return 0;
            }
            else {
                if(s[i-2] == '0') dp[i] = dp[i-1];
                else if(s[i-2]<'2' || (s[i-2]=='2' && s[i-1]<='6'))
                    dp[i] = dp[i-1] + dp[i-2];
                else dp[i] = dp[i-1];
            }
        }

        return dp[s.length()];
    }
};
```
