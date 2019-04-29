---
title: Leetcode-Roman to Integer
updated: 2019-04-09 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Roman numerals are represented by seven different symbols: `I`, `V`, `X`, `L`, `C`, `D` and `M`.

<!-- more -->

```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

For example, two is written as `II` in Roman numeral, just two one's added together. Twelve is written as, `XII`, which is simply `X` + `II`. The number twenty seven is written as `XXVII`, which is `XX` + `V` + `II`.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not `IIII`. Instead, the number four is written as `IV`. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as `IX`. There are six instances where subtraction is used:

- `I` can be placed before `V` (5) and `X` (10) to make 4 and 9. 
- `X` can be placed before `L` (50) and `C` (100) to make 40 and 90. 
- `C` can be placed before `D` (500) and `M` (1000) to make 400 and 900.

Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

**Example 1:**

```
Input: "III"
Output: 3
```

**Example 2:**

```
Input: "IV"
Output: 4
```

**Example 3:**

```
Input: "IX"
Output: 9
```

**Example 4:**

```
Input: "LVIII"
Output: 58
Explanation: L = 50, V= 5, III = 3.
```

**Example 5:**

```
Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

最开始想用每种情况都写一遍，虽然会的不多，但还是嫌麻烦，想到了Python中的用两个`list`去对应相同的索引的不同类型，所以用数组老对应不同的罗马符号。

**Solution：**

```
class Solution {
public:
    int romanToInt(string s) {
        int num[] = {1, 5, 10, 50, 100, 500, 1000};
        string rom = "IVXLCDM";
        vector<int> tmp;
        int res = 0;
        for (int i = 0; i < s.size(); i++)
        {
            for (int j = 0; j < 7; j++)
            {
                if (s[i] == rom[j])
                    tmp.push_back(num[j]);
            }
        }
        for (int i = 0; i < tmp.size(); i++)
        {
            if (i == tmp.size() - 1 || tmp[i] >= tmp[i + 1])
            {
                res += tmp[i];
            }
            else
            {
                res -= tmp[i];
            }
        }
        return res;
    }
};
```

`if (i == tmp.size() - 1 || tmp[i] >= tmp[i + 1])`，这里最开始是`if(tmp[i] >= tmp[i + 1])`，处理`"MCDLXXVI"`时报错了，输出结果发现最后的`I`并没有加进去，很神奇，原因还没找到，先留个坑。

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| ----------------- | ------------------------------------------------------------ | ------- | ------ | -------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/218498486/) | 20 ms   | 9 MB   | cpp      |

