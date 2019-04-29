---
title: Leetcode-Remove Duplicates from Sorted Array
updated: 2019-04-17 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Given a sorted array *nums*, remove the duplicates [**in-place**](https://en.wikipedia.org/wiki/In-place_algorithm) such that each element appear only *once* and return the new length.

Do not allocate extra space for another array, you must do this by **modifying the input array in-place** with O(1) extra memory.

<!-- more -->

**Example 1:**

```
Given nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively.

It doesn't matter what you leave beyond the returned length.
```

**Example 2:**

```
Given nums = [0,0,1,1,1,2,2,3,3,4],

Your function should return length = 5, with the first five elements of nums being modified to 0, 1, 2, 3, and 4 respectively.

It doesn't matter what values are set beyond the returned length.
```

**Clarification:**

Confused why the returned value is an integer but your answer is an array?

Note that the input array is passed in by **reference**, which means modification to the input array will be known to the caller as well.

Internally you can think of this:

```
// nums is passed in by reference. (i.e., without making a copy)
int len = removeDuplicates(nums);

// any modification to nums in your function would be known by the caller.
// using the length returned by your function, it prints the first len elements.
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

**Solution:**
`Vector`在日常写程序用的比较多，所以看一眼就有了点想法。
传入的是`vector`的引用，需要在原数组上操作，`vector`已经排序了，比较索引为`i`和`i+1`的元素是否相同就可以，因为美股元素只能出现一次，所以有两种解决办法，遇到相同的元素移到最后或者删除，移到最后还要额外增加一个变量来计算不同元素的数量，所以直接用删除的方式。
需要注意的就是`vector`的删除问题，当删除索引为`i`的元素时，`i`后面的元素都会向前移动一位，要注意`i++`的索引的变化。

```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if (nums.size() == 0) {
            return 0;
        }
        for (int i = 0; i < nums.size() - 1; i++) {
            if (nums[i] == nums[i + 1]) {
                nums.erase(nums.begin() + i + 1);
                i--; //注意索引的变化
            }
        }  
        return nums.size();
    }
};
```

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| :---------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/224381666/) | 156 ms  | 9.9 MB | cpp      |

大神的solution：

```C++
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        nums.erase(std::unique(nums.begin(), nums.end()), nums.end());
        return nums.size();
    }
};
```

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| :---------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/224392614/) | 24 ms   | 9.8 MB | cpp      |