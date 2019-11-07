---
title: Leetcode-Median of Two Sorted Arrays
date: 2019-11-07 20:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
mathjax: true
---

数据分布都不明显，让机器怎么学。

<!-- more -->

### Problem

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

**Example:**

There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

You may assume **nums1** and **nums2** cannot be both empty.

**Example 1:**

```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0
```

**Example 2:**

```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

### Process

+ 合并两个有序的数组
+ 找到数组的中位数

**std::meger**

```c
template <class InputIterator1, class InputIterator2, class OutputIterator>
  OutputIterator merge (InputIterator1 first1, InputIterator1 last1,
                        InputIterator2 first2, InputIterator2 last2,
                        OutputIterator result)
{
  while (true) {
    if (first1==last1) return std::copy(first2,last2,result);
    if (first2==last2) return std::copy(first1,last1,result);
    *result++ = (*first2<*first1)? *first2++ : *first1++;
  }
}
```

**Parameters**

- first1, last1

  [Input iterators](http://www.cplusplus.com/InputIterator) to the initial and final positions of the first sorted sequence. The range used is `[first1,last1)`, which contains all the elements between first1 and last1, including the element pointed by first1 but not the element pointed by last1.

- first2, last2

  [Input iterators](http://www.cplusplus.com/InputIterator) to the initial and final positions of the second sorted sequence. The range used is `[first2,last2)`.

- result

  [Output iterator](http://www.cplusplus.com/OutputIterator) to the initial position of the range where the resulting combined range is stored. Its size is equal to the sum of both ranges above.

- comp

  Binary function that accepts two arguments of the types pointed by the iterators, and returns a value convertible to `bool`. The value returned indicates whether the first argument is considered to go before the second in the specific *strict weak ordering* it defines. 

  The function shall not modify any of its arguments. 

  This can either be a function pointer or a function object. 

  **Return value**

  An iterator pointing to the *past-the-end* element in the resulting sequence.

### Solution

```c++
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        
        vector<int> merged;
        double median;
        merge(nums1.begin(), nums1.end(),
              nums2.begin(), nums2.end(),
              back_inserter(merged));

        if (merged.size() % 2 == 0) {
            median = (double)(merged[merged.size() / 2 - 1] + merged[merged.size() / 2]) / 2;
        }
        else {
            median = merged[(merged.size() - 1) / 2];
        }
        
        return median;
    }
};
```

### Result

| Time Submitted    | Status                                                       | Runtime | Memory  | Language |
| :---------------- | :----------------------------------------------------------- | :------ | :------ | :------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/276758080/) | 24 ms   | 10.5 MB | cpp      |

### Summary

TODO：Solution的解法如何实现。