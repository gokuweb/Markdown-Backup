---
title: Python 赋值、浅拷贝和深拷贝
categories:
  - 代码
tags:
  - python
abbrlink: e58ae62f
date: 2019-05-24 09:32:02
---

&#8195;&#8195;Python 赋值确切的说是 **对象的引用**，赋值的过程就是修改内存地址指向的过程，而非像 C 语言等修改内存地址中的数据。如果学过其他语言的多重赋值，那你可能也会对  Python 的 **多重赋值** 有点疑惑，Python 的多重赋值中各变量之间不会相互影响，也跟赋值顺序无关，比如 `a, b = 1, a+b` 和 `b, a = a+b, 1` 结果是一样的，先对 a 赋值也不会影响到 b。
<!-- more -->

## python 赋值

### 普通赋值
&#8195;&#8195;C 语言是先给一个定义了的变量分配内存空间，赋值的时候直接把数据存储到这个内存地址，python 是系统先给对象分配内存空间，然后将变量指向这个地址，一个修改数据，一个修改地址指向。先来对比下 python 和 C 中变量的内存地址：
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
&#8195;&#8195;可以看到 C 中分配了两个不同的内存地址，但他们存放的值都是 1，对 a 和 b 的修改就是修改对应的数据，地址不变。

```python
#python 赋值

if __name__ == '__main__':
    a = 1
    b = a
    print(id(a), id(b))
    
# 140735081374352
# 140735081374352
```
&#8195;&#8195;可以看到 python 中 只分配了一个内存空间，a 和 b 是同一地址，系统先给 1 这个对象分配了内存空间，然后把 a 指向这个地址，再让 b 指向 a 所指的对象。既然 a 和 b 的地址一样，那对 a 重新赋值后 b 的值会变吗？这取决于对象的类型，python 中分为可变对象（mutable object）和不可变对象（immutable object），数值类型（int 和 float）、字符串 str、元组 tuple 都是不可变对象，而列表 list、字典 dict、集合 set 都是可变对象。

&#8195;&#8195;**两个可变对象不管是否相同，都有独立的内存地址；两个不可变对象如果相同，它们共同指向一个内存地址：**
```python
if __name__ == '__main__':
    a = ["hello", ["w", "o", "r", "l", "d"]]
    b = ["hello", ["w", "o", "r", "l", "d"]]
    c = "hello"
    d = "hello"

    print(id(a), id(b))
    print(id(c), id(d))
    print(id(a[0]), id(a[1]), id(a[1][1]))
    print(id(b[0]), id(b[1]), id(b[1][1]))

```
&#8195;&#8195;看下各变量的值和地址：

| 变量 | 值 | 地址 |
|--|--|--|
| a | ['hello', ['w', 'o', 'r', 'l', 'd']] | 2221522424200 |
| b | ['hello', ['w', 'o', 'r', 'l', 'd']] | 2221523093576 |
| c | <font color="blue">'hello'</font> | <font color="blue">2221523076184</font> |
| d | <font color="blue">'hello'</font> | <font color="blue">2221523076184</font> |
| a\[0\] | <font color="blue">'hello'</font> | <font color="blue">2221523076184</font> |
| a\[1\] | <font color="red">['w', 'o', 'r', 'l', 'd']</font> | <font color="red">2221522425416</font> |
| a\[1\]\[1\] | 'w' | 2221521479080 |
| b\[0\] | <font color="blue">'hello'</font> | <font color="blue">2221523076184</font> |
| b\[1\] | <font color="red">['w', 'o', 'r', 'l', 'd']</font> | <font color="red">2221522628552</font> |
| b\[1\]\[1\] | 'w' | 2221521479080 |

&#8195;&#8195;可以看出不可变对象值相同那么它们地址也相同，可变对象即使值相同地址也不同。当然如果用 `b = a` 形式的赋值，不管是什么对象，它们的地址肯定是一样的。

### 不可变对象
&#8195;&#8195;一个不可变对象在创建后就不能被改变：
```python
if __name__ == '__main__':
    a = 3
    b = a
    b = 5
    print("a:", a)
      
# a: 3
```
&#8195;&#8195;前两个赋值让 a 和 b 都指向了不可变对象 3，当执行 `b = 5` 时并不能把 3 变为 5，而是让 b 又指向了 5，虽然结果和 C 一样，但原理不同，C 更新的是内存单元中的值，而 python 是更新了地址的指向，如下图：
![immutable object](/imgs/immutable-object.png)

&#8195;&#8195;python 中数字是不可变对象，所谓自增就是自身增加，不可变对象无法自增，所以 **python 中没有 C 语言的自增（++）和自减（--）运算符**，它只能从一个对象指向下一个对象，可以这样写 a += 1。

### 可变对象
&#8195;&#8195;可变对象在创建后可以被修改：
```python
if __name__ == '__main__':   
    a = [1, 2, 3]
    b = a
    b[0] = 1024
    print("a:", a)
    
# a: [1024, 2, 3]
```
&#8195;&#8195;前两个赋值让 a 和 b 都指向了可变对象 `[1, 2, 3]`，当执行 `b[0] = 1024` 时会修改对象的第一个元素，所以 a 和 b 指向的都成了 `[1024, 2, 3]`。不是说 python 赋值改变的是地址指向吗，怎么又可以修改对象了？其实这里的修改对象并非是把某一内存空间中的 1 变成了 1024，而是把原本指向 1 的地址指向了 1024，对于 a 来说它的第一个元素值变了，但对于 a[0] 来说，只是改变了 a[0] 的指向，如下图：
![mutable object1](/imgs/mutable-object1.png)


&#8195;&#8195;再看一个例子：
```python
if __name__ == '__main__':
    values = [0, 1, 2]
    values[1] = values
    print(values)
    
# [0, [...], 2]
```
&#8195;&#8195;我们预想的结果是 `[0, [0, 1, 2], 2]`，但实际上是 `[0, [...], 2]`，首先 `values` 指向了对象 `[0, 1, 2]`，接着又把 `values` 的第二个元素指向 `values` 所引用的对象：
![mutable object3](/imgs/mutable-object3.png)

&#8195;&#8195;如果想要复制一个列表，就要用到拷贝了：
```python
if __name__ == '__main__':
    values = [0, 1, 2]
    values[1] = values[:]
    #values[1] = values.copy()
    print(values)

# [0, [0, 1, 2], 2]
```

## python 拷贝
&#8195;&#8195;python 中有 **浅拷贝** 和 **深拷贝** 两种：
> 1. 浅拷贝(copy)：拷贝父对象，不会拷贝对象的内部的子对象。
> 2. 深拷贝(deepcopy)： copy 模块的 deepcopy 方法，完全拷贝了父对象及其子对象。

赋值：
![python-assign](/imgs/python-assign.png)

浅拷贝（shallow copy）：
![python-copy1](/imgs/python-copy1.png)

深拷贝（deep copy）：
![python-deepcopy1](/imgs/python-deepcopy1.png)

### 浅拷贝
&#8195;&#8195;**浅拷贝是指创建一个新的对象，其内容是原对象中元素的引用**，浅拷贝可以用 `copy` 或者 `slice` 实现：
```python
>>> a = [1, 2, 3]
>>> b = list(a)
>>> print(id(a), id(b))          # a和b身份不同
140601785066200 140601784764968
>>> for x, y in zip(a, b):       # 但它们包含的子对象身份相同
...     print(id(x), id(y))
... 
140601911441984 140601911441984
140601911442016 140601911442016
140601911442048 140601911442048
```
&#8195;&#8195;可以看到 a 和 b 指向内存中不同的 list 对象，但它们的元素却指向相同的地址。使用浅拷贝，当列表对象有嵌套的时候也会产生出乎意料的错误，我们仅想修改 a 列表，结果 b 列表也被修改：
```python
if __name__ == '__main__':
    a = [0, [1, 2], 3]
    b = a[:]
    c = a
    a[0] = 8
    a[1][1] = 9
    print(a, b, c)
    
# [8, [1, 9], 3] 
# [0, [1, 9], 3]
# [8, [1, 9], 3]
```
&#8195;&#8195;浅拷贝创建了新对象 `b`，`b` 中有三个元素但这三个元素都是 `a` 元素的引用，所以 `a` 和 `b` 的地址不同，但 `a[0], b[0]`、`a[1], b[1]`、`a[2], b[2]` 分别指向相同的地址，对 `a[0], a[1], a[2]` 的修改只是改变了自己的指向，`b[0], b[1], b[2]` 依旧指向之前的地址，没什么影响，但是对 `a[1]` 或者 `b[1]` **内部的元素** 做修改，就不一样了。
&#8195;&#8195;`a[1]` 本身是一个列表，是可变对象，对 `a[1][0]` 和 `a[1][1]` 做修改后 `a[1]` 本身的地址并没变，`b[1]` 又和 `a[1]` 指向相同的地址，所以 `b[1][0]` 和 `b[1][1]` 也会产生相应的变化，**因此对组合对象（比如列表中包含列表）的拷贝要慎重**。
![python-copy2](/imgs/python-copy2.png)

### 深拷贝
&#8195;&#8195;**深拷贝会把父对象子对象都复制一份到新的内存空间**，想要复制嵌套元素最好是进行"深拷贝"（deep copy），需要引入 `copy` 模块：
```python
import copy

if __name__ == '__main__':
    a = [0, [1, 2], 3]
    b = copy.deepcopy(a)
    a[0] = 8
    a[1][1] = 9
    print(a, "\n", b)
    
```
&#8195;&#8195;效果如下图，内存空间中父对象子对象都会有两份：
![python-deepcopy2](/imgs/python-deepcopy2.png)

### 对比
&#8195;&#8195;似乎浅拷贝和深拷贝并没有什么区别？a 和 b 的地址不同但它们中的元素地址相同，a 和 c 的地址不同但它们中的元素地址也相同。
```python
>>> import copy
>>> a = [1, 2, 3]
>>> b = list(a)                   # 浅拷贝得到b
>>> c = copy.deepcopy(a)          # 深拷贝得到c
>>> print(id(a), id(b))           # a 和 b 不同
140601785066200 140601784764968
>>> for x, y in zip(a, b):        # a 和 b 的子对象相同
...     print(id(x), id(y))
... 
140601911441984 140601911441984
140601911442016 140601911442016
140601911442048 140601911442048

>>> print(id(a), id(b))           # a 和 b 不同
140601785065840 140601785066200
>>> for x, y in zip(a, b):        # a 和 b 的子对象相同
...     print(id(x), id(y))
... 
140601911441984 140601911441984
140601911442016 140601911442016
140601911442048 140601911442048
```

&#8195;&#8195;这是因为 a 的元素都是不可变对象，对于 **不可变对象**，当需要一个新的对象时，python 可能会返回已经存在的某个类型和值都一致的对象的引用。而且这种机制并不会影响 a 和 b 的相互独立性，因为当两个元素指向同一个不可变对象时，对其中一个赋值不会影响另外一个。来看一下 **可变对象** 的例子：
```python
>>> import copy
>>> a = [[1, 2],[5, 6], [8, 9]]
>>> b = copy.copy(a)              # 浅拷贝得到b
>>> c = copy.deepcopy(a)          # 深拷贝得到c
>>> print(id(a), id(b))           # a 和 b 不同
139832578518984 139832578335520
>>> for x, y in zip(a, b):        # a 和 b 的子对象相同
...     print(id(x), id(y))
... 
139832578622816 139832578622816
139832578622672 139832578622672
139832578623104 139832578623104

>>> print(id(a), id(c))           # a 和 c 不同
139832578518984 139832578622456
>>> for x, y in zip(a, c):        # a 和 c 的子对象也不同
...     print(id(x), id(y))
... 
139832578622816 139832578621520
139832578622672 139832578518912
139832578623104 139832578623392
```
&#8195;&#8195;所以浅拷贝和深拷贝仅仅是对组合对象也就是可变对象来说的，所谓的组合对象就是包含了其它对象的对象，如列表，类实例等。而对于数字、字符串以及其它“原子”类型，没有拷贝一说，产生的都是原对象的引用。

## python 多重赋值
&#8195;&#8195;多重赋值、元组解包、迭代解包指的都是同一件事情，多重赋值实际上是创建一个元组，然后循环遍历该元组，并从循环中获取每个 items ，再分别赋值给变量，下面代码是等价的：
```python
x, y = 10, 20
x, y = (10, 20)
(x, y) = 10, 20
(x, y) = (10, 20)
```

&#8195;&#8195;python 的多重赋值语句中，各变量之间不会相互影响，也跟赋值顺序无关，看代码：
```python
if __name__ == '__main__':
    a = 3
    a, b = 1, a
    print(a, b)
    
# 1, 3
```
&#8195;&#8195;在 python 中输出的是 `1, 3`，并非 `1, 1`，也就是对 `a` 的新赋值并没有影响到对 `b` 的操作，这跟 C 或者其他高级语言不同，在 python 的 **多重赋值过程中** ，变量之间不会相互产生影响，但是会影响到下一次使用这些变量。下面是一个单链表逆序的例子：
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

## 二维数组初始化
&#8195;&#8195;假如要创建一个 4x3 的二维数组，有两种方法：

** list * n 形式（不推荐）**

```python
>>> b = [[0] * 3] * 4
[[0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]

>>> print(id(b[0]), id(b[1]), id(b[3]))
2524376102984 2524376102984 2524376102984

>>> b[0][1] = 5
[[0, 5, 0], [0, 5, 0], [0, 5, 0], [0, 5, 0]]
```
&#8195;&#8195;**n 个 list 的浅拷贝的连接**，`b[1], b[2], b[3]` 都是 `b[0]` 的浅拷贝，它们指向的是同一地址，而且又是可变对象，所以它们中任一个的内部元素被修改，都会影响到其他。一维数组没事。

**for 循环赋值**

&#8195;&#8195;通过循环赋初始值是一种比较推荐的初始化方法，不会因为对象引用的问题产生 bug。
```python
>>> b = [[0 for _ in range(3)] for _ in range(4)]
[[0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]

>>> print(id(b[0]), id(b[1]), id(b[3]))
2326272240328 2326272238088 2326272375624

>>> b[0][1] = 5
[[0, 5, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]]
```

## 参考

[Python连续赋值需要注意的地方](https://blog.csdn.net/xjcvip007/article/details/54348245)
[如何理解 Python 的赋值逻辑](http://www.cnblogs.com/andywenzhi/p/7453374.html)
[python 深入理解 赋值、引用、拷贝、作用域](https://www.cnblogs.com/jiangzhaowei/p/5740913.html)
[Python FAQ2：赋值、浅拷贝、深拷贝的区别？](https://songlee24.github.io/2014/08/15/python-FAQ-02/)
[python——赋值与深浅拷贝](https://www.cnblogs.com/Eva-J/p/5534037.html)
[Python 直接赋值、浅拷贝和深度拷贝解析](https://www.runoob.com/w3cnote/python-understanding-dict-copy-shallow-or-deep.html)
[（译）用多重赋值和元组解包提高python代码的可读性](https://juejin.im/post/5aace6746fb9a028b61749dc)
[Python多维数组初始化的两种方式和浅拷贝问题](https://blog.csdn.net/haha_point/article/details/78320673)
[python的二维数组操作](https://www.cnblogs.com/btchenguang/archive/2012/01/30/2332479.html)

