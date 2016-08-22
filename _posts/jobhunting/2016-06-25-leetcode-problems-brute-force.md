---
layout: post
title: LeetCode Problems - DFS & BFS & 枚举
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

## 暴力枚举

### 17 - Letter Combinations of a Phone Number

```c++
class Solution {
public:
    vector<string> letterCombinations(string digits) {
        char T[][8] = {"abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

        vector<string> coms;
        if(digits.size() ==0 || digits[0] == '0' || digits[0] == '1') return coms;

        int count = (digits[0]=='7'||digits[0]=='9') ? 4 : 3;

        if(digits.size() == 1) {
            for(int i=0;i<count;++i) {
                string tmp = "";
                tmp.append(1, T[digits[0]-'2'][i]);
                coms.push_back(tmp);
            }
        }
        else {
            vector<string> restComs = letterCombinations(digits.substr(1, digits.size()-1));
            for(int i=0;i<count;++i) {
                for(int j=0;j<restComs.size();++j) {
                    coms.push_back(T[digits[0]-'2'][i] + restComs[j]);
                }
            }
        }

        return coms;
    }
};
```

## DFS

### 77 - combinations

```c++
class Solution {
public:
    vector<vector<int>> combine(int n, int k) {
        vector<vector<int>> ans;
        vector<int> comb;
        foo(ans, comb, 1, n,k);
        return ans;
    }

    void foo(vector<vector<int>> &ans, vector<int> comb, int index, int n, int k) {
        if(comb.size() == k) {
            ans.push_back(comb);
            return;
        }

        for(int i=index;i<=n;++i) {
            comb.push_back(i);
            foo(ans, comb, i+1, n, k);
            comb.pop_back();
        }
    }
};
```

### 39 - Combination Sum

```c++
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> ans;
        sort(candidates.begin(), candidates.end());
        vector<int> com;
        f(ans, candidates, com, 0, target);
        return ans;
    }

    void f(vector<vector<int>> &ans, vector<int> cand, vector<int> com, int index, int target){
        if(!target){
            ans.push_back(com);
            return;
        }

        for(int i=index;i<cand.size() && target>=cand[i];++i){
            com.push_back(cand[i]);
            f(ans, cand, com, i, target-cand[i]);
            com.pop_back();
        }
    }
};
```

## BFS
