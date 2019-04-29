---
title: Leetcode-Reverse Integer
updated: 2019-04-06 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Given a 32-bit signed integer, reverse digits of an integer.

<!-- more -->

**Example 1:**

```
Input: 123
Output: 321
```

**Example 2:**

```
Input: -123
Output: -321
```

**Example 3:**

```
Input: 120
Output: 21
```

**Note:**
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: $[−2^{31},  2^{31 − 1}]$. For the purpose of this problem, assume that your function returns 0 when the reversed integer overflows.

 看起来是一个特别简单的问题，第一想法也是个野路子，先把整型转字符串，再将字符串逆序排列一下，在转换回整型，感觉不妥。

算法思想特别简单，

```
res = res * 10 + x % 10;
```

其实我没想到，看了别人的答案才领悟。

这里很容易懂，但是数据溢出是个大问题，我以为一句

```
x >= INT_MIN || x <= INT_MAX
```

就可以解决了，too young, too naive. 果然要偷偷看答案了。

**核心思想：**

1. pop = x % 10, If $temp = res\times10 + pop$ causes overflow, then it must be that $res\geq\frac{INTMAX}{10}$
2. If  $res\geq\frac{INTMAX}{10}$, then  $temp = res\times10 + pop$ is guaranteed to overflow
3. If $res==\frac{INTMAX}{10}$, then  $temp = res\times10 + pop$ will overflow if and only if  $pop > 7$

```c++
class Solution {
public:
    int reverse(int x) {
        int rev = 0;
        while (x != 0) {
            int pop = x % 10;
            x /= 10;
            if (rev > INT_MAX/10 || (rev == INT_MAX / 10 && pop > 7)) return 0;
            if (rev < INT_MIN/10 || (rev == INT_MIN / 10 && pop < -8)) return 0;
            rev = rev * 10 + pop;
        }
        return rev;
    }
};
```

