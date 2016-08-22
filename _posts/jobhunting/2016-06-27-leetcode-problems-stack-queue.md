---
layout: post
title: LeetCode Problems - Stack and Queue
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

## Stack

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
