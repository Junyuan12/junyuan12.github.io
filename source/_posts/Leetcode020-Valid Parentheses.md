---
title: Leetcode-Valid Parentheses
updated: 2019-04-12 12:00:00
categories: Leetcode
tags:
     - C++
     - algorithm
---

之前就看到这道题，没想到好的方法，正好最近在搞数据结构，到栈这里一个应用就是括号的匹配。

Given a string containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['` and `']'`, determine if the input string is valid.

<!-- more -->

An input string is valid if:

1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

Note that an empty string is also considered valid.

**Example 1:**

```
Input: "()"
Output: true
```

**Example 2:**

```
Input: "()[]{}"
Output: true
```

**Example 3:**

```
Input: "(]"
Output: false
```

**Example 4:**

```
Input: "([)]"
Output: false
```

**Example 5:**

```
Input: "{[]}"
Output: true
```

**Solution:**

```
class Solution {
public:
    bool isValid(string str) {
    stack<char> s1;
    for (int i = 0; i < str.size(); i++) {
        if (str[i] == '(' || str[i] == '[' || str[i] == '{') {
            s1.push(str[i]);
        }
        else if (str[i] == ')' || str[i] == ']' || str[i] == '}') {
            if (s1.empty()) {
                return false;
            }
            else if ((s1.top()=='(' && str[i]==')') || (s1.top()=='[' && str[i]==']') || (s1.top()=='{' && str[i]=='}')) {
                s1.pop();
            }
            else {
                return false;
            } 
        }
    }

    if (s1.empty())
        return true;
    else
        return false;
    }
};
```

数据结构果然很重要啊！！！

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| ----------------- | ------------------------------------------------------------ | ------- | ------ | -------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/221560125/) | 4 ms    | 8.4 MB | cpp      |