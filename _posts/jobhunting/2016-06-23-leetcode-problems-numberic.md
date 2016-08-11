---
layout: post
title: LeetCode Problems - Numberic
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

## 整数表示与转换

### 12 - Integer to Roman

```c++
class Solution {
public:
    string intToRoman(int num) {
        string T[4][10] = {{"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"},
                           {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"},
                           {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"},
                           {"", "M", "MM", "MMM"}};
        string result;

        if(num >= 1000) { result += T[3][num/1000]; num = num%1000; }
        if(num >= 100) { result += T[2][num/100]; num = num%100; }
        if(num >= 10) { result += T[1][num/10]; num = num%10; }
        if(num >= 1) result += T[0][num];

        return result;
    }
};
```

### 13 - Roman to Integer

```c++
class Solution {
public:
    int romanToInt(string s) {
        int num = 0;

        int i = 0;
        while(i<s.length() && s[i]=='M') { num += 1000; ++i; }
        if(i+1<s.length() && s[i]=='C' && s[i+1]=='D') { num += 400; i+=2; }
        if(i+1<s.length() && s[i]=='C' && s[i+1]=='M') { num += 900; i+=2; }
        while(i<s.length() && s[i]=='D') { num += 500; ++i; }
        while(i<s.length() && s[i]=='C') { num += 100; ++i; }
        if(i+1<s.length() && s[i]=='X' && s[i+1]=='L') { num += 40; i+=2; }
        if(i+1<s.length() && s[i]=='X' && s[i+1]=='C') { num += 90; i+=2; }
        while(i<s.length() && s[i]=='L') { num += 50; ++i; }
        while(i<s.length() && s[i]=='X') { num += 10; ++i; }
        if(i+1<s.length() && s[i]=='I' && s[i+1]=='V') { num += 4; i+=2; }
        if(i+1<s.length() && s[i]=='I' && s[i+1]=='X') { num += 9; i+=2; }
        while(i<s.length() && s[i]=='V') { num += 5; ++i; }
        while(i<s.length() && s[i]=='I') { num += 1; ++i; }

        return num;
    }
};
```

### 273 - Integer to English Words

```c++
class Solution {
public:
    string numberToWords(int num) {
        if(num == 0) return "Zero";

        return bar(0, num);
    }

    string bar(int i, int num) {
        if(num <= 0) return "";

        char T[][16] = {"", "Thousand", "Million", "Billion"};

        string highRes = bar(i+1, num/1000);
        string lowRes = foo(num%1000);

        string blank = " ";

        if(highRes.length()>0 && lowRes.length()>0)
            return highRes + " " + lowRes + (i<=0 ? "" : blank + T[i]);

        if(highRes.length() > 0)
            return highRes;

        if(lowRes.length() > 0)
            return lowRes + (i<=0 ? "" : blank + T[i]);

        return "";
    }

    string foo(int num) {
        char T[][16] = {"", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", "Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"};
        char T2[][16] = {"", "", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};

        string result = "";

        if(num >= 100) {
            result += T[num/100]; result += " Hundred";
            num = num%100;
        }

        if(num >= 20) {
            if(result.length() > 0) result += " ";
            result += T2[num/10];
            num = num%10;
        }

        if(num >= 1) {
            if(result.length() > 0) result += " ";
            result += T[num];
        }

        return result;
    }
};
```

## 整数计算

## 二进制计算
