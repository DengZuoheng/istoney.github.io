---
layout: post
title: Google APAC Problem Solutions
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
    ifstream in("A-large-practice.in");
	ofstream out("A-large-practice.out");

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

> 
