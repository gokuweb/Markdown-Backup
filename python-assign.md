---
title: Python 的赋值和拷贝
categories:
  - 代码
tags:
  - python
abbrlink: e58ae62f
date: 2019-05-24 09:32:02
---

&#8195;&#8195;C 语言的赋值会开辟新的内存地址，然后存放对应数据，Python 是把一切数据都看成对象，为每一个对象分配一个内存空间，所谓的赋值就是让变量指向某一个对象，C 语言赋值变得是内存地址中的值，python 赋值变得是地址的指向。其实我觉得比较有意思的是 python 的 **连续赋值**，连续赋值语句中的各变量之间不会相互影响，也跟赋值顺序无关，比如 `a, b = 1, a+b` 和 `b, a = a+b, 1` 结果是一样的，先对 a 赋值也不会影响到 b。
<!-- more -->

## python 赋值

### 变量的内存地址
```C
#C 语言赋值
#include <stdio.h>

int main(){
    int a = 1;
    int b = a;
    printf("%p\n%p", &a,&b);
}

# 0x7fff349522e8
# 0x7fff349522ec
```
&#8195;&#8195;可以看到 C 中分配了两个不同的内存地址，但他们存放的值都是 1。

```python
#python 赋值

if __name__ == '__main__':
    a = 1
    b = a
    print(id(a), id(b))
    
# 140735081374352
# 140735081374352
```
&#8195;&#8195;可以看到 python 中 a 和 b 是同一内存地址，其实 python 中的赋值确切的说是 **对象的引用**，`b = a` 只是让 b 也指向 a 所指的对象，所以 a 和 b 都指向 1。python 中一切都是对象，分为可变对象（mutable object）和不可变对象（immutable object），数值类型（int 和 float）、字符串 str、元组 tuple 都是不可变对象，而列表 list、字典 dict、集合 set 都是可变对象。

### 不可变对象
```python
if __name__ == '__main__':
    a = 3
    b = a
    b = 5
    print("a:", a)
      
# a: 3
```
&#8195;&#8195;前两个赋值让 a 和 b 都指向了不可变对象 3，当执行 `b = 5` 时并不能把 3 变为 5，而是让 b 又指向了 5，虽然结果和 C 一样，但原理不同，C 更新的是内存单元中存放的值，而 Python 更新的是变量的指向，如下图：
![assign](/imgs/assign.png)

&#8195;&#8195;python 中数字是不可变对象，所谓自增就是自身增加，不可变对象无法自增，所以 **python 中没有 C 语言的自增（++）和自减（--）运算符**，它只能从一个对象指向下一个对象，可以这样写 a += 1。

### 可变对象
```python
if __name__ == '__main__':   
    a = [1, 2, 3]
    b = a
    b[0] = 1024
    print("a:", a)
    
# a: [1024, 2, 3]
```
&#8195;&#8195;前两个赋值让 a 和 b 都指向了可变对象 `[1, 2, 3]`，当执行 `b[0] = 1024` 时会修改对象的第一个元素，所以 a 和 b 指向的都成了 `[1024, 2, 3]`，如下图：
![assignobject](/imgs/assignobject.png)

&#8195;&#8195;再看一个例子：
```python
if __name__ == '__main__':
    values = [0, 1, 2]
    values[1] = values
    print(values)
    
# [0, [...], 2]
```
&#8195;&#8195;我们预想的结果是 `[0, [0, 1, 2], 2]`，但实际上是 `[0, [...], 2]`，首先 `values` 指向了对象 `[0, 1, 2]`，接着又把 `values` 的第二个元素指向 `values` 所引用的对象：
![assign-list](/imgs/assign-list.png)

&#8195;&#8195;如果想要复制一个列表，就要用到拷贝了。

## python 拷贝
&#8195;&#8195;python 中有 **浅拷贝** 和 **深拷贝** 两种：
> 1. 浅拷贝(copy)：拷贝父对象，不会拷贝对象的内部的子对象。
> 2. 深拷贝(deepcopy)： copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

&#8195;&#8195;浅拷贝可以用 `copy` 或者 `slice` 实现：
```python
if __name__ == '__main__':
    values = [0, 1, 2]
    values[1] = values[:]
    #values[1] = values.copy()
    print(values)
    
# [0, [0, 1, 2], 2]
```
&#8195;&#8195;使用浅拷贝时，当列表对象有嵌套的时候也会产生出乎意料的错误，比如：
```python
if __name__ == '__main__':
    a = [0, [1, 2], 3]
    b = a[:]
    a[0] = 8
    a[1][1] = 9
    print(a, "\n", b)
    
# [8, [1, 9], 3] 
# [0, [1, 9], 3]
```
&#8195;&#8195;a 和 b 是一个独立的对象，但他们的子对象还是指向统一对象：
![assign-list2](/imgs/assign-list2.png)

&#8195;&#8195;想要复制嵌套元素的方法是进行"深复制"（deep copy），需要引入 `copy` 模块：
```python
import copy

if __name__ == '__main__':
    a = [0, [1, 2], 3]
    b = copy.deepcopy(a)
    a[0] = 8
    a[1][1] = 9
    print(a, "\n", b)
    
```
效果如下图：
![assign-list3](/imgs/assign-list3.png)

## python 连续赋值
&#8195;&#8195;python 的连续赋值语句中，各变量之间不会相互影响，也跟赋值顺序无关，看代码：
```python
if __name__ == '__main__':
    a = 3
    a, b = 1, a
    print(a, b)
    
# 1, 3
```
&#8195;&#8195;在 python 中输出的是 `1, 3`，并非 `1, 1`，也就是对 `a` 的新赋值并没有影响到对 `b` 的操作，这跟 C 或者其他高级语言不同，在 python 的 **连续赋值过程中** ，变量之间不会相互产生影响，但是会影响到下一次使用这些变量。下面是一个单链表逆序的例子：
```python
class ListNode:
    def __init__(self, x):
        self.val = x
        self.next = None

def reverseList(head):
    L = ListNode(float("-inf"))
    print(L.val)
    while head:
        L.next, head.next, head = head, L.next, head.next
    return L.next
    
def l3_print(L3):
    while(L3):
        print(L3.val)
        if L3.next is not None:
            L3 = L3.next
        else:
            break
            
if __name__ == '__main__':
    l1_1 = ListNode(1)
    l1_2 = ListNode(2)
    l1_3 = ListNode(3)
    l1_4 = ListNode(4)
    l1_1.next = l1_2
    l1_2.next = l1_3
    l1_3.next = l1_4
    l3 = reverseList(l1_1)
    l3_print(l3)
```

## 参考

[Python连续赋值需要注意的地方](https://blog.csdn.net/xjcvip007/article/details/54348245)
[如何理解 Python 的赋值逻辑](http://www.cnblogs.com/andywenzhi/p/7453374.html)
[python 深入理解 赋值、引用、拷贝、作用域](https://www.cnblogs.com/jiangzhaowei/p/5740913.html)
