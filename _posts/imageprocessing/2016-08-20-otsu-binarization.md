---
layout: post
title: 图像二值化之OTSU（最大类间方差法，大津法）
category: Image Processing
tags: [image, otsu, binarization]
---
{% include JB/setup %}

## 概述

最大类间方差法是日本学者大津展之（Nobuyuki Otsu）于1979年提出的一种自适应阈值计算算法，又称为大津法，简称为OTSU[[1]](https://en.wikipedia.org/wiki/Otsu%27s_method)。该算法根据像素的灰度值，将灰度图片划分为前景和背景两个类别。通过计算两个类别的类间方差（intra-class variance）来判断前景和背景差异的显著程度。并且通过搜索使类间方差达到最优的类别划分界限作为最优阈值。

## 原理

设灰度图片大小为w\*h，即图片的像素个数为w\*h. 设类别划分阈值为threshold，则灰度值小于threshold的所有像素为前景，大于threshold的所有像素为背景。设前景像素个数的占比为\\( \omega_0 \\)，其平均灰度为\\( \mu_0 \\)，背景像素个数占比为\\( \omega_1 \\)，平均灰度为\\( \mu_1 \\)。图片整体平均灰度为\\( \mu \\)。则，

\\[ \omega_0 + \omega_1 = 1 \\]
\\[ \mu = \omega_0*\mu_0 + \omega_1*\mu_1 \\]

可得类间方差为

\\[ variance = \omega_0*(\mu_0-\mu)^2 + \omega_1*(\mu_1-\mu)^2 \\]

带入上述两个等式，可得

\\[ variance = \omega_0\omega_1*(\mu_0 - \mu_1)^2 \\]

## 实现

计算时通过遍历图片灰度的值空间，计算每个阈值对应的类间方差，比较求得最优阈值。因为类间方差的值只是用来比较，所以计算中分别使用像素个数代替即可。C++实现代码如下。

```c++
const int GREY_MAX_VALUE = 255;

int getOtsuThreshold(int[] grey, int w, int h) {
  int n = GREY_MAX_VALUE + 1;
  int histogram[n];
  fill(histogram, histogram+n, 0);
  for(int i=0;i<w*h;++i) {
    ++histogram[grey[i] & 0xFF];
  }
  
  int total = w*h, count0 = 0, count1 = 0;
  int sum_total, sum0 = 0, sum1 = 0;
  for(int i=0;i<n;++i) {
    sum_total += i*histogram[i];
  }
  
  double mean0, mean1, max_var = 0;
  int t = 0, threshold;
  while(t < n) {
    count0 += histogram[t];
    count1 = total - count0;
    
    if(count0 == 0) continue;
    if(count1 == 0) break;
    
    sum0 += t*histogram[t];
    sum1 = sum_total - sum0;
    
    mean0 = (double) sum0 / count0;
    mean2 = (double) sum1 / count1;
    
    double tmp = count0*count1*(mean0 - mean1)*(mean0 - mean1);
    if(tmp > max_var) {
      max_var = tmp;
      threshold = t;
    }
  }
  
  return threshold;
}
```

