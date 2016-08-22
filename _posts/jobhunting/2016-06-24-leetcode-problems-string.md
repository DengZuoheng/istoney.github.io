---
layout: post
title: LeetCode Problems - String 字符串
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

### Unclassified

### 6 - ZigZag Conversion

```c++
class Solution{
public:
    string convert(string s, int numRows){
        if(numRows <= 1) return s;

        string res;
        int n = numRows*2 - 2;
        int size=s.length();

        int src, step, step2;
        for(int i=0;i<numRows;i++){
            src = i;
            if(i==0 || i==numRows-1){
                step = n;
                while(src<size){
                    res.push_back(s[src]);
                    src += step;
                }
            } else {
                step = (numRows-1-i)*2;
                step2 = n-step;
                bool f = 1;
                while(src<size){
                    res.push_back(s[src]);
                    src += f ? step : step2;
                    f = !f;
                }
            }
        }
        return res;
    }
};
```

### 14 - Longest Common Prefix

```c++
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(strs.size() == 0) return "";

        string prefix = strs[0];

        for(int i=1;i<strs.size();++i) {
            int j;
            for(j=0;j<prefix.size() && j<strs[i].size() && prefix[j]==strs[i][j];++j) {}

            prefix.resize(j);
            if(j == 0) break;
        }

        return prefix;
    }
};
```

## 回文

### 5 - Longest Palindromic Substring

```c++
class Solution{
public:
    string longestPalindrome(string s){
        string s2;
        for(int i=0;i<s.length();i++){
            s2.push_back('#');
            s2.push_back(s[i]);
        }
        s2.push_back('#');

        int max=0, mid, len=0, index=0;
        int p[2010];
        for(int i=0;i<s2.length();i++){
            if(max > i){
                p[i] = min(p[mid*2-i], max-i);
            } else p[i] = 1;

            while(i-p[i]>=0 && i+p[i]<s2.length() && s2[i-p[i]]==s2[i+p[i]]){
                p[i]++;
            }

            if(i+p[i] > max){
                max = i+p[i];
                mid = i;
            }

            if(p[i] > len){
                index = i;
                len = p[i];
            }
        }

        //#a#b#a# 3 4
        if(index & 0x1) return s.substr(index/2-(len-1)/2, len-1);
        //#a#b#b#a# 4 5
        else return s.substr(index/2-(len-1)/2, len-1);
    }
};
```
