---
title: Leetcode-Add Two Numbers
date: 2019-05-25 15:06:00
categories: Leetcode
tags:
     - C++
     - algorithm
mathjax: true
---

本打算好好看看数据结构，结果看到递归后就被一些乱七八糟的事一直拖着，一直也没往下看，而且隔段时间不回去看一下，就容易忘了，所以心血来潮，挑战了了第一道`Medium`，其实也没想的那么难，只不过考虑的东西要比`Easy`的多一些而已，然而，能踩的坑我还是都踩了一遍。

<!-- more -->

### Problem

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example:**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

给了两个非空的单链表，每一个单链表作为一个倒序的十进制的数，对这两个十进制数求和，得到的结果**倒序输出**，既然同样倒序放到一个新的列表中，其实这样就没那么复杂了，既然都是倒序，那就从头开始做，先从单链表的第一个元素，即十进制数对应的个位数开始计算，只需要考虑进位的问题就可以了。

### Process

+ 两个单链表的长度可能不同，如果不同，在短的后面补零；

  **Note：**对两个同时判断，否则长度相同时会死循环。

+ 对同一个位置的数求和$ (sum) $，如果$sum > 10$，则对$ sum $取余，进位数加$ 1 $；

+ 当两个链表的`next`都为空时就意味着加法结束了，如果进位数$ > 0 $，则结果再增加一位最高位，并设置为$ 1 $。

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
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        ListNode* res = new ListNode(0);
        ListNode* ptr = res;
        int sum, carry = 0;
        
        while (l1 || l2) {
            if (l1->next == NULL && l2->next != NULL) {
                l1->next = new ListNode(0);
            }
            if (l2->next == NULL&& l1->next != NULL) {
                l2->next = new ListNode(0);
            }
             sum = l1->val + l2->val + carry;
            
             carry = 0;
             if (sum >= 10) {
                 sum = sum % 10;
                 carry = 1;
             }    
             ptr->next = new ListNode(sum);
             ptr = ptr->next;
             if (l1->next == NULL && l2->next == NULL && carry > 0) {
                 ptr->next = new ListNode(1);
                 l1 = l1->next;
                 l2 = l2->next;
             }
             else {
                 l1 = l1->next;
                 l2 = l2->next;
             }
         }
        ptr = res->next;
        delete res;
        
        return ptr;
    }
};
```

### Better Solution

```C++
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
        int sum=0;
        ListNode *l3=NULL;
        ListNode **node=&l3;
        while(l1!=NULL||l2!=NULL||sum>0)
        {
            if(l1!=NULL)
            {
                sum+=l1->val;
                l1=l1->next;
            }
            if(l2!=NULL)
            {
                sum+=l2->val;
                l2=l2->next;
            }
            (*node)=new ListNode(sum%10);
            sum/=10;
            node=&((*node)->next);
        }        
        return l3;
    }
};
```

思路其实没什么区别，但是这样写很明显比我的更好理解。

### Result

| Time Submitted    | Status                                                       | Runtime | Memory | Language |
| ----------------- | ------------------------------------------------------------ | ------- | ------ | -------- |
| a few seconds ago | [Accepted](https://leetcode.com/submissions/detail/221560125/) | 4 ms    | 8.4 MB | cpp      |

### Summary

数据结构和语言的学习要一直在路上了，这一道题，加踩坑加磨叽，用了好长时间，实在不应该。

p.s. 每次在学校的理发店剪完头都想和理发师拼命啊有木有！！！