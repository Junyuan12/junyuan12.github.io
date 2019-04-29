---
title: Leetcode-Two Sum
date: 2019-03-28 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the same element twice.

<!-- more -->

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

**Solution:**

我的解决方案：

```
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> index;
        for (int i = 0; i < nums.size(); i++)
        {
            for (int j = i; j < nums.size(); j++)
            {
                if (i != j && (nums[i] + nums[j] == target))
                {
                    index.push_back(i);
                    index.push_back(j);
                }
            }
        }
        return index;
    }
};
```

```
Runtime: 384 ms, faster than 5.00% of C++ online submissions for Two Sum.
Memory Usage: 9.4 MB, less than 90.00% of C++ online submissions for Two Sum.
```

算法比较蠢，单纯的遍历，长路漫漫，还要好好学。

Top Voted Solution：

The basic idea is to maintain a hash table for each element num in nums, using num as key and its index (0-based) as value. For each num, search for target - num in the hash table. If it is found and is not the same element as num, then we are done.

```
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> indices;
        for (int i = 0; i < nums.size(); i++) {
            if (indices.find(target - nums[i]) != indices.end()) {
                return {indices[target - nums[i]], i};
            }
            indices[nums[i]] = i;
        }
        return {};
    }
};
```

