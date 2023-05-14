---
title: 跟着AI学Python(day1)
date: 2023-04-05 21:22:49
tags:
  - Python
categories: Python
---

### 介绍
刚学习完 mosh 的 Python 6小时完整课程，想要通过实践去提升自己的代码能力从而应用在工作当中，以下是我通过Microsoft的NewBing去获取Python中的练习题，并尝试自行解决，然后提交给Newbing检查代码。

跟着AI学习Python，让编程变得更加轻松和有趣。

### 练习
在这个学习过程中，我会跟随AI助教，每天完成AI出的三道题目，一步一步地掌握Python的基础知识和实用技巧。在实践中，我将花费30分钟的时间进行独立完成（可以Google，完成后交与AI进行批改，查看代码改进的地方，从而记录下来。以下我将会给出自己写的代码，与AI的代码, 和需要改进的地方。

#### 题目一
写一个函数，它接受一个字符串作为参数，然后返回这个字符串的反转。例如，如果输入"hello"，则返回"olleh"。
{% tabs first %}
<!-- tab self code -->
```python
def number():  
    print(x[::-1])

number()
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
def reverse_string():               # 函数名最好能够反映出它们的功能
    x = "string"
    print(x[::-1])

reverse_string()
``` 
<!-- endtab -->
{% endtabs %}

#### 题目二
写一个函数，它接受一个列表作为参数，然后返回这个列表中的最大值和最小值。例如，如果输入[1, 2, 3, 4, 5]，则返回(5, 1)。
{% tabs second %}
<!-- tab self code -->
```python
def list():
    l = ["1", "2", "3", "4", "5"]        
    max_l = max(l)
    min_l = min(l)
    return max_l, min_l

print(list())
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
def list():
    #你的list函数中的l列表最好用数字而不是字符串，因为字符串的比较是按照字典序的，可能会出现意外的结果。比如"10"会比"2"小，因为"1"的ASCII码比"2"小。
    l = [1, 2, 3, 4, 5]        
    max_l = max(l)
    min_l = min(l)
    return max_l, min_l

print(list())

``` 
<!-- endtab -->
{% endtabs %}

#### 题目三
写一个函数，它接受一个正整数n作为参数，然后返回斐波那契数列的第n项。斐波那契数列是这样定义的:
第一项和第二项都是1，从第三项开始，每一项都是前两项的和。例如，斐波那契数列的前10项是：1, 1, 2, 3, 5, 8, 13, 21, 34, 55。
{% tabs frist %}
<!-- tab self code -->
```python
None
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
# 用递归实现
def fibonacci_recursive(n):
    # 如果n是1或者2，直接返回1
    if n == 1 or n == 2:
        return 1
    # 否则，返回前两项的和
    else:
        return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2)

# 用循环实现
def fibonacci_iterative(n):
    # 初始化第一项和第二项为1
    a = 1
    b = 1
    # 如果n是1或者2，直接返回1
    if n == 1 or n == 2:
        return 1
    # 否则，从第三项开始循环计算
    else:
        # 循环n-2次
        for i in range(n - 2):
            # 计算当前项为前两项的和
            c = a + b
            # 更新前两项为当前项和前一项
            a = b
            b = c
        # 返回最后一项
        return c

# 测试代码
print(fibonacci_recursive(10))
print(fibonacci_iterative(10))
``` 
<!-- endtab -->
{% endtabs %}