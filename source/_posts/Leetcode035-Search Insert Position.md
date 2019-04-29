---
title: Leetcode-Search Insert Position
updated: 2019-04-29 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

<!-- more -->

You may assume no duplicates in the array.

**Example 1:**

```
Input: [1,3,5,6], 5
Output: 2
```

**Example 2:**

```
Input: [1,3,5,6], 2
Output: 1
```

**Example 3:**

```
Input: [1,3,5,6], 7
Output: 4
```

**Example 4:**

```
Input: [1,3,5,6], 0
Output: 0
```

**Solution:**

```c++
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int res = 0;
        if (nums.size() == 0 || nums[0] > target) {
            return 0;
        }
        else if (nums.back() < target) {
            return nums.size();
        }
        else {
            for (int i = 0; i < nums.size() - 1; i++) {
                if (nums[i] == target) {
                    res = i;
                }
                if (nums[i] < target && nums[i+1] >= target) {
                    res = i+1;
                }
            }
            return res;
        }
    }
};
```

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| :---------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/225636350/) | 8 ms    | 8.9 MB | cpp      |

```
Runtime: 8 ms, faster than 98.61% of C++ online submissions for Search Insert Position.
Memory Usage: 8.9 MB, less than 98.08% of C++ online submissions for Search Insert Position.
```

查找的部分应该还可以优化，用二分查找做还可以再快一点。