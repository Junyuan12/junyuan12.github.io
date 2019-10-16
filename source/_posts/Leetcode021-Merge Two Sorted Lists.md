---
title: Leetcode-Merge Two Sorted Lists
date: 2019-10-16 18:52:00
categories: Leetcode
tags:
     - C++
     - algorithm
mathjax: true
---

天黑有伞，下雨有灯。

<!-- more -->

### Problem

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

**Example:**

```
Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```

### Process

+ 遍历两个链表，比较元素的值
+ 设置一个值，指向当前的节点，方便向合并的链表中增加元素
+ 最后将剩余的值放到最后面

### Solution

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        
        if (l1 == NULL) {
            return l2;
        }
        
        if (l2 == NULL) {
            return l1;
        }
        
        ListNode *l3, *res;
        
        if (l1->val <= l2->val) {
            l3 = new ListNode(l1->val);
            l1 = l1->next;
        }
        else {
            l3 = new ListNode(l2->val);
            l2 = l2->next;
        }
        
        res = l3;
        
        while (l1 != NULL && l2 != NULL) {
            if (l1->val <= l2->val) {
                 l3->next = l1;
                 l1 = l1->next;
             }
            else {
                l3->next = l2;
                l2 = l2->next;
            }
            l3 = l3->next;
        }
        if (l1 != NULL) {
            l3->next = l1;
        }
        if (l2 != NULL) {
            l3->next = l2;
        }
        
        return res;
    }
};
```

### Result

| Time Submitted | Status                                                       | Runtime | Memory | Language |
| :------------- | :----------------------------------------------------------- | :------ | :----- | :------- |
| 2 minutes ago  | [Accepted](https://leetcode.com/submissions/detail/270390606/) | 8 ms    | 8.7 MB | cpp      |

### Summary

没什么特别的地方，做的还是稀碎。