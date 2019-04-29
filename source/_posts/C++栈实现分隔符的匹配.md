---
title: C++栈实现分隔符的匹配
updated: 2019-04-14 12:00:00
categories: C++
tags:
     - C++
     - algorithm
---

栈是一种后进先出(*LIFO*)的线性数据结构。

<!-- more -->

栈适用于数据存储后以相反的顺序来检索的情况。站的一个应用是在程序中匹配分隔符。

在C++程序中存在下列分隔符：圆括号"`(`"和"`)`"、方括号"`[`"和"`]`"、花括号"`{`"和"`}`"、注释分隔符"`/*`"和"`*/`"

**栈的操作：**

+ clear()——清空栈。
+ isEmpty()——判断栈是否为空。
+ push(el)——将元素el放到栈的顶部。
+ pop()——弹出栈顶部的元素。
+ topEL()——获取栈顶部的元素，不删除。

**genStack.h**

```
//******************* genStack.h ************************
//   generic class for vector implenmentation of stack

#ifndef STACK
#define STACK

#include <vector>

using namespace std;

template<class T, int capacity = 30>
class Stack {
public:
    Stack() {
        pool.reserve(capacity);
    }

    void clear() {
        pool.clear();
    }

    bool isEmpty() const {
        return pool.empty();
    }

    T& topEl() {
        return pool.back();
    }

    T pop() {
        T el = pool.back();
        pool.pop_back();
        return el;
    }

    void push(const T& el) {
        pool.push_back(el);
    }

private:
    vector<T> pool;
};

#endif
```

**genStack.cpp**

```
//******************* genStack.h ************************
//用栈实现分隔符的匹配

#include <iostream>
#include "genStack.h"
#include <string>

using namespace std;

int main() {
    string str = "s=t[5]+u/(v*(w+y))";
    Stack<char> s1;
    for (int i = 0; i < str.size(); i++) {
        //注意单引号和双引号的使用！！！
        if (str[i] == '(' || str[i] == '[' || str[i] == '{') {
            s1.push(str[i]);
            //cout << "topEL: " << s1.topEl() << endl;
        }
        else if (str[i] == ')' || str[i] == ']' || str[i] == '}') {
            if ((s1.topEl()=='(' && str[i]==')') || (s1.topEl()=='[' && str[i]==']') || (s1.topEl()=='{' && str[i]=='}')) {
                s1.pop();
            }
            else {
                cout << "error!" << endl;
                system("pause");
                return 0;
            } 
        }
        else if (str[i] == '/') {
            if (str[i + 1] == '*') {
                for (int j = i; j < str.size(); j++) {
                    if (str[j] == '*' && str[j + 1] == '/') {
                        i = j + 1;
                        break;
                    }
                    else if (j == str.size()-1) {
                        cout << "'/*' can not match '*/'" << endl;
                        system("pause");
                        return 0;
                    }
                }
            }
            else
                continue;
        }
    }

    if (s1.isEmpty())
        cout << "success!" << endl;
    else
        cout << "failed!" << endl;
    system("pause");
    return 0;
}
```

