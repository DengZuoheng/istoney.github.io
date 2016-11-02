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

## 括号匹配

### 20 - Valid Parentheses

```c++
class Solution {
public:
    bool isValid(string s) {
        stack<char> S;

        for(int i=0;i<s.length();++i) {
            if(s[i]=='(' || s[i]=='[' || s[i]=='{')
                S.push(s[i]);
            else if(s[i]==')' && !S.empty() && S.top()=='(')
                S.pop();
            else if(s[i]==']' && !S.empty() && S.top()=='[')
                S.pop();
            else if(s[i]=='}' && !S.empty() && S.top()=='{')
                S.pop();
            else return false;
        }

        return S.empty();
    }
};
```

### 22 - Generate Parentheses

```c++
class Solution {
public:
    vector<string> generateParenthesis(int n) {
        vector<string> ans;

        if(n > 0)
            foo(ans, "", n, n);

        return ans;
    }

    void foo(vector<string>& ans, string str, int restLeftNum, int restRightNum) {
        if(restLeftNum==0 && restRightNum==0) ans.push_back(str);

        if(restLeftNum > 0)
            foo(ans, str+"(", restLeftNum-1, restRightNum);

        if(restRightNum > restLeftNum)
            foo(ans, str+")", restLeftNum, restRightNum-1);
    }
};
```

```c++
class Solution {
public:
    int longestValidParentheses(string s) {
        int start = 0;

        int leftCount = 0, rightCount = 0;

        int maxLen = 0;

        for(int i=0;i<s.length();++i) {
            if(s[i] == '(') ++leftCount;
            else if(leftCount > rightCount)
                ++rightCount;
            else {
                maxLen = max(maxLen, foo(s, start, i));
                start = i+1;
            }
        }

        maxLen = max(maxLen, foo(s, start, s.length()));

        return maxLen;
    }

    int foo(const string &s, int start, int end) {
        int st = end - 1;
        int rightCount = 0, leftCount = 0;
        int maxLen = 0;

        for(int i=end-1;i>=start;--i) {
            if(s[i] == ')') ++rightCount;
            else if(rightCount > leftCount)
                ++leftCount;
            else {
                maxLen = max(maxLen, st-i);
                st = i - 1;
                rightCount = leftCount = 0;
            }
        }

        maxLen = max(maxLen, st-start+1);

        return maxLen;
    }
};
```
