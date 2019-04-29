---
title: Leetcode-Longest Common Prefix
date: 2019-04-11 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string `""`.

<!-- more -->

**Example 1:**

```
Input: ["flower","flow","flight"]
Output: "fl"
```

**Example 2:**

```
Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```

**Note:**

All given inputs are in lowercase letters `a-z`.

**Solution：**

```C++
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.size() == 0)
            return "";
        int minlen = INT_MAX;
        string res = "";
        for (int i = 0; i < strs.size(); i++)
        {
            if (strs[i].size() < minlen)
                minlen = strs[i].size();
        }
        for (int i = 0; i < minlen; i++)
        {
            bool flag = true;
            for (int j = 0; j < strs.size() - 1; j++)
            {
                if (strs[j][i] != strs[j + 1][i])
                    flag = false;
            }
            if (flag == true)
                res = res + strs[0][i];
            else
                break;
        }
        
        return res;
    }
};
```

没什么亮点的写法。

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| ----------------- | ------------------------------------------------------------ | ------- | ------ | -------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/218924015/) | 8 ms    | 9.1 MB | cpp      |

