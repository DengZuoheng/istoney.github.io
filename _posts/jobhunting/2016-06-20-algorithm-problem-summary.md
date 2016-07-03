---
layout: post
title: Algorithm Problems Summary
category: Job Hunting
tags: [Job]
---
{% include JB/setup %}

### 数组
LeetCode #1 - Two Sum `easy` <br>
> 给定整数数组，返回和为目标值的两个数的下标，结果唯一. <br>

1. 排序后双指针夹逼，因为结果为下标，排序时需要将下标跟随值异同排序，复杂度$O(nlogn)$；
2. Hash表记录每个数及下标，遍历查找，复杂度$O(n)$.

LeetCode #15 - 3Sum `Medium` <br>
> 给定整数数组，返回所有和为0的三元组，不得重复.
