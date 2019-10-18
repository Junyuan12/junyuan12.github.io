---
title: Leetcode-Longest Substring Without Repeating Characters
date: 2019-10-11 12:40:00
categories: Leetcode
tags:
     - C++
     - algorithm
mathjax: true
---

金九已过，还在挣扎。

<!-- more -->

### Problem

Given a string, find the length of the **longest substring** without repeating characters.

**Example 1:**

```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
```

**Example 2:**

```
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```

**Example 3:**

```
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

题干要理解清楚，**要找到最长的不重复的子序列**，不是从重复的位置重新统计，踩坑了。

### Process

+ 从第二个元素开始，只要与已有的序列不重复，序列长度加1
+ 如果重复，比较当前序列与上一个序列的长度，留下长度大的
+ 反向遍历当前序列，从没有重复的位置开始继续保存序列
+ 输出最长的序列的长度

### Solution

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        
        vector<char> subString;
        int length = 0;
        vector<char>::iterator iter;
        vector<char>::reverse_iterator riter;
        
        for (int i = 0; i < s.size(); i++) {
            if (subString.empty()) {
                subString.push_back(s[i]);
            }
            iter = find(subString.begin(), subString.end(), s[i]);
            if(iter != subString.end()) { //s[i] in subString
                if (subString.size() > length) {
                    length = subString.size();
                }
                // 反向迭代vector，找到s[i]之前与s[i]不相等的元素
                int index = 0;
                for(riter = subString.rbegin(); riter != subString.rend();++riter)
                {
                    if (*riter == s[i]) {
                        break;
                    } 
                    index++;
                }
                int delete_length = subString.size() - index;
                subString.erase(subString.begin(), subString.begin() + delete_length);
                subString.push_back(s[i]);
            } 
            else {
                subString.push_back(s[i]);
                if (subString.size() > length) {
                    length = subString.size();
                }
            }
        }
        return length;
    }
};
```

### Better Solution

```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int left = 0, len = 0;
        map<char, int> dict;
        for (int right=0; right<s.size(); right++) {
            if (dict.find(s[right]) != dict.end()) {
                left = max(left, dict[s[right]] + 1);
            }
            dict[s[right]] = right;
            len = max(len, right - left + 1);
        }
        return len;
    }
};
```

### Result

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| :---------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/268837409/) | 40 ms   | 9.3 MB | cpp      |
| 2 minutes ago     | [Wrong Answer](https://leetcode.com/submissions/detail/268836788/) | N/A     | N/A    | cpp      |
| 40 minutes ago    | [Wrong Answer](https://leetcode.com/submissions/detail/268826526/) | N/A     | N/A    | cpp      |
| 44 minutes ago    | [Wrong Answer](https://leetcode.com/submissions/detail/268825655/) | N/A     | N/A    | cpp      |
| an hour ago       | [Compile Error](https://leetcode.com/submissions/detail/268812576/) | N/A     | N/A    | cpp      |

### Summary

最开始想到的方法是栈，后来查找元素的时候发现用栈不太方便；后来换了vector，题没审明白，找到重复的元素直接清空了vector，导致结果出错。

从不重复的元素开始统计，正序遍历vector容易忽略后面存在的重复的元素，所以反向遍历vector，找到第一个重复的元素，然后删除其实元素到该重复元素，继续找不重复的序列，输出最长子序列的长度。

感觉用双端队列可能会稍微简单点吧，就不存在反向遍历的操作了。

better solution只是先拷过来，map操作还是不太能看得懂。