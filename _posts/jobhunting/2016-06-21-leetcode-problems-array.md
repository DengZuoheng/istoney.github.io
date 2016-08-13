---
layout: post
title: LeetCode Problems - Array
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

## Unclassified

### 26 - Remove Duplicates from Sorted Array

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int index = 1;
        for(int i=1;i<nums.size();++i)
            if(nums[i] != nums[i-1])
                nums[index++] = nums[i];

        if(index < nums.size())
            nums.resize(index);

        return nums.size();
    }
};
```

### 80 - Remove Duplicates from Sorted Array II

```c++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        int index = 1;
        int status = 1; // in case nums[0]==nums[1]
        for(int i=1;i<nums.size();++i) {
            if(nums[i] != nums[i-1]) {
                nums[index++] = nums[i];
                status = 1;
            }
            else if(status == 1) {
                nums[index++] = nums[i];
                status = 2;
            }
        }

        if(index < nums.size())
            nums.resize(index);

        return nums.size();
    }
};
```

## 求和

## 面积

### 11 - Container With Most Water

```c++
class Solution {
public:
    int maxArea(vector<int>& height) {
        int i = 0, j = height.size()-1;
        int maxArea = 0;

        while(i < j) {
            int area = min(height[i], height[j]) * (j-i);
            maxArea = max(maxArea, area);

            if(height[i] < height[j]) ++i;
            else --j;
        }

        return maxArea;
    }
};
```

### 42 - Trapping Rain Water

```c++
class Solution {
public:
    int trap(vector<int>& height) {
        int maxHeightLeft[height.size()];
        int maxHeightRight[height.size()];

        int maxHeight = 0;
        for(int i=0;i<height.size();++i)
            maxHeightLeft[i] = maxHeight = max(maxHeight, height[i]);
        maxHeight = 0;
        for(int i=height.size()-1;i>=0;--i)
            maxHeightRight[i] = maxHeight = max(maxHeight, height[i]);

        int water = 0;
        for(int i=0;i<height.size();++i) {
            int tmp = min(maxHeightLeft[i], maxHeightRight[i]) - height[i];
            water += tmp>0 ? tmp : 0;
        }

        return water;
    }
};
```

### 84 - Largest Rectangle in Histogram

```c++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int lowHeightLeft[heights.size()];
        int lowHeightRight[heights.size()];

        lowHeightLeft[0] = -1;
        for(int i=1;i<heights.size();++i) {
            int j = i - 1;
            while(heights[j]>=heights[i] && j>=0)
                j = lowHeightLeft[j];
            lowHeightLeft[i] = j;
        }

        lowHeightRight[heights.size()-1] = heights.size();
        for(int i=heights.size()-2;i>=0;--i) {
            int j = i + 1;
            while(heights[j]>=heights[i] && j<heights.size())
                j = lowHeightRight[j];
            lowHeightRight[i] = j;
        }

        int maxRect = 0;
        for(int i=0;i<heights.size();++i) {
            //cout<<lowHeightRight[i]<<" "<<lowHeightLeft[i]<<endl;
            int rect = heights[i] * (lowHeightRight[i] - lowHeightLeft[i] - 1);
            maxRect = max(maxRect, rect);
        }

        return maxRect;
    }
};
```

## Stock

### 121 - Best Time to Buy and Sell Stock

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.empty()) return 0;

        int curtMin = prices[0];
        int maxProfit = 0;

        for(int i=1;i<prices.size();++i) {
            maxProfit = max(maxProfit, prices[i]-curtMin);
            curtMin = min(curtMin, prices[i]);
        }

        return maxProfit;
    }
};
```

### 122 - Best Time to Buy and Sell Stock II

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.empty()) return 0;

        bool bought = false;
        int buyPrice;
        int maxProfit = 0;

        for(int i=1;i<prices.size();++i) {
            if(prices[i] > prices[i-1] && !bought) {
                bought = true;
                buyPrice = prices[i-1];
            }

            if(bought && (i+1==prices.size() || prices[i]>prices[i+1])) {
                bought = false;
                maxProfit += prices[i] - buyPrice;
            }
        }

        return maxProfit;
    }
};
```

### 123 - Best Time to Buy and Sell Stock III

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if(prices.empty()) return 0;

        int firstMaxProfit[prices.size()];
        int secondMaxProfit[prices.size()];

        firstMaxProfit[0] = 0;
        int minPrice = prices[0];
        int maxProfit = 0;

        for(int i=1;i<prices.size();++i) {
            firstMaxProfit[i] = maxProfit = max(maxProfit, prices[i]-minPrice);
            minPrice = min(minPrice, prices[i]);
        }

        secondMaxProfit[prices.size()-1] = 0;
        int maxPrice = prices[prices.size()-1];
        maxProfit = 0;

        for(int i=prices.size()-2;i>=0;--i) {
            secondMaxProfit[i] = maxProfit = max(maxProfit, maxPrice-prices[i]);
            maxPrice = max(maxPrice, prices[i]);
        }

        maxProfit = secondMaxProfit[0];
        for(int i=0;i<prices.size()-1;++i)
            maxProfit = max(maxProfit, firstMaxProfit[i] + secondMaxProfit[i+1]);
        maxProfit = max(maxProfit, firstMaxProfit[prices.size()-1]);

        return maxProfit;
    }
};
```
