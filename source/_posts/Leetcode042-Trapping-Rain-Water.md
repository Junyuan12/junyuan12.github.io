---
title: Leetcode042-Trapping Rain Water
date: 2019-10-22 20:44:58
categories: Leetcode
tags:
     - C++
     - algorithm
---

> 努力，奋斗。

<!-- more -->

**Note: ** 这是一个标准的错误答案，但是思路我觉得还可以，只是记录一下，方便日后学习。

还以为自己想到了一个不错的方法，现实又是一记重拳。

### Problem

 Given *n* non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining. 

![](Leetcode042-Trapping-Rain-Water\rainwatertrap.png)

 *The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped. *

**Example:**

```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

题读起来很简单，就是将黑色部分看成一个容器，计算容器中雨水（蓝色块）的数量。

### Process

**思路1：**暴力方法，就是从前往后一点点比较，但是设置的变量越来越多，用的有点懵， 而且最后那一部分出来起来实在是闲麻烦，突然就想到一层一层统计了。

**思路2：**每层是否有雨水，肯定与容器的高度有关，可以统计每层雨水的数量，只要两边被块包围，中间为空的地方肯定就是有雨水的；所以按照框的长度和最高的高度创建一个容器，根据每列数组的大小设置每列中1的数量，最后分层处理，也就是一行一行统计，求和得到雨水的水量。

没想到内存炸了。

### Solution

**Note: ** 这个解法内存占用没通过测试。

```c
class Solution {
public:
    int trap(vector<int>& height) 
    {
        int water = 0;
        int max = 0;
        
        for (int val: height)
        {
            if (val > max)
            {
                max = val;
            }
        }
        
        // 创建二维数组
        vector <vector<int> > rain_arr;
        rain_arr.resize(max);
        for(int i = 0 ; i < max; i++)
	    {
		    rain_arr[i].resize(height.size());
	    }
            
        // 二维数组赋值
        for (int i = 0; i < max; i++)
        {
            for (int j = 0; j < height.size(); j++)
            {
                rain_arr[i][j] = 0;
            }
        }
        // 根据每个bar的大小为每一列添加bar个1
        // 注意索引的变化
        for (int i = 0; i < height.size(); i++)
        {
            for (int j = 0; j < height[i]; j++)
            {
                rain_arr[j][i] = 1;
            }
        }
        
        for (int i = 0; i < max; i++)
        {
            for (int j = 0; j < height.size(); j++)
            {
                cout << rain_arr[i][j] << " ";
            }
            cout << endl;
        }
        
        for (int i = 0; i < max; i++)
        {
            // 这里怎么求起始1和结束1之间0的个数有问题
            // 暂时用vector了
            vector <int> index_one;
            for (int j = 0; j < height.size(); j++)
            {
                if (rain_arr[i][j] == 1)
                {
                    index_one.push_back(j);
                }
            }
            // cout << index_one.back() << ", " << index_one.front() << ", " << index_one.size() << endl;
            // front和back是第一个元素和最后一个元素的引用，不是begin和end
            water += index_one.back() - index_one.front() + 1 - index_one.size();  
        }
               
        return water;
    }
};
```

### Result

| Time Submitted | Status                                                       | Runtime | Memory | Language |
| :------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| 7 minutes ago  | [Memory Limit Exceeded](https://leetcode.com/submissions/detail/272213495/) | N/A     | N/A    | cpp      |

> **314 / 315** test cases passed.

 证明思路还是对的。

### Summary

兜兜转转，坏消息好消息轮着来，好在可能离算法近了一步；常师兄说话和张老师好像，感觉他俩的性格就很合适一起合作。