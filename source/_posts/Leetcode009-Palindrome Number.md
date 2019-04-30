---
title: Leetcode-Palindrome Number
date: 2019-04-07 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

<!-- more -->

**Example 1:**

```c++
Input: 121
Output: true
```

**Example 2:**

```c++
Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.
```

**Example 3:**

```c++
Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

**Follow up:**

Coud you solve it without converting the integer to a string?

算法思想和第7题差不多，而且说了不让用string，就是第7题之后加个判断，感觉自己在偷懒，但是还是想了好长时间才想出来。

**Solution：**

```c++
class Solution {
public:
    bool isPalindrome(int x) {
        int res = 0, tmp = x;
        if (x < 0)
            return false;
        while (tmp != 0) {
            int pop = tmp % 10;
            tmp /= 10;
            if (res > INT_MAX/10 || (res == INT_MAX / 10 && pop > 7)) return 0;
            if (res < INT_MIN/10 || (res == INT_MIN / 10 && pop < -8)) return 0;
            res = res * 10 + pop;
        }
        
        if (res == x)
            return true;
        else
            return false;
    }
};
```

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| ----------------- | ------------------------------------------------------------ | ------- | ------ | :------: |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/218192437/) | 32 ms   | 8 MB   |   cpp    |

最近负能量有点多，需要心灵鸡汤透一透，最好是喝完能上头的那种。