---
title: Python 总结
categories:
  - 算法
tags:
  - python
abbrlink: c3c2983d
date: 2019-06-27 20:51:34
---
&#8195;&#8195;整理记录一些 Python 的基础知识，比如切片负步长的操作、空集合的定义、`is` 和 `==` 的区别、`None` 和 `if not` 的区别、`for` 迭代过程中修改列表等，持续更新。。。
<!-- more -->

## slice

### 描述

&#8195;&#8195;在 Python 中，有序列结构的数据都可以使用切片操作，比如 **列表、元组、字符串** 等，字典是无序的所以不可以。切片操作产生一个和原对象 **同类型** 的副本，是一种 **浅拷贝** （详细了解参考 [Python 赋值、浅拷贝和深拷贝](https://www.gokuweb.com/algorithm/e58ae62f.html)），列表的切片还是列表，元组的切片仍是元组。

**语法：**

> str[stop]
> str[start: stop: step]



**参数说明：**

> start -- 起始位置
> stop -- 结束位置
> step -- 间距


**返回值：**

> 返回一个切片对象。


&#8195;&#8195;切片是根据元素编号或者说索引进行操作的，做逆序处理时候经常用到负数，所以每个元素都有对应的正负编号。范围区间是 **左闭右开** ，而且 **可以越界** ，假设 str = '0123456789′ ：

&#8195;&#8195;当 step > 0 时，位置编号为从左边的 0 为起点开始，往右依次递增（0,1,2,3,4,5...），如下图所示：

![str1](\imgs\str1.png)

> 例：str[::1] 。
>
> 解析：默认 start = 0，end = len(str)，step = 1，所以等同于 str[0, 10, 1] ，第 10 位取不到。
>
> 结果：0 1 2 3 4 5 6 7 8 9


&#8195;&#8195;当 step < 0 时，位置编号为从右边的 -1 为起点开始，从右往左依次递减（...，-5，-4，-3，-2，-1），如图：

![str2](\imgs\str2.png)

> 例：str[::-1] 。
>
> 解析：默认 start = -1，end = -(len(str) + 1)，step = -1，所以等同于 str[-1, -11, -1] ，第 -11 位取不到。
>
> 结果：9 8 7 6 5 4 3 2 1 0



&#8195;&#8195;其实 start 和 end 缺省情况下默认是 None，我也没看过源码，但是从测试结果来看上面的分析应该是合理的，也许这个 None 并不是确切的上下界，而是一个足够大的上下界？来看几个 None 的万能用法：

```python
>>> str = "0123456789"
>>> str[None:10:1]
'0123456789'
>>> str[0:None:1]
'0123456789'
>>> str[None:None:1]
'0123456789'

>>> str[-1:None:-1]
'9876543210'
>>> str[None:-11:-1]
'9876543210'
>>> str[None:None:-1]
'9876543210'
```



### 逆序截取

&#8195;&#8195;从 Python 1.4 开始才有了扩展切片（Extended Slices），引入了第三个参数 step，当 step = -1 时原字符串会被直接逆序，但是如果只想截取部分字符串进行逆序，就需要注意一些细节。

**当 step > 0 时，begin 必须在 end 左边，即 begin < end**

&#8195;&#8195;因为 step > 0 时候是从左往右遍历的，begin < end 才是有效的区间，当不符合情况时，没有符合的数据会返回空：

```python
>>> str = "0123456789"
>>> str[2:6:1]
'2345'
>>> str[0:10:2]
'02468'

>>> str[6:2:1]
''
```



**当 step < 0 时，begin 必须在 end 右边，即 begin > end**

&#8195;&#8195;因为 step < 0 时候是从右往左遍历的，begin > end 才是有效的区间，当不符合情况时，没有符合的数据会返回空：

```python
>>> str = "0123456789"
>>> str[-2:-6:-1]
'8765'
>>> str[-1:-11:-2]
'97531'

>>> str[-6:-2:-1]
''
```



&#8195;&#8195;range 也适用这个规则：

```python
>>> x = [i for i in range(2,5,1)]
>>> x
[2, 3, 4]

>>> x = [i for i in range(-2,-5,-1)]
>>> x
[-2, -3, -4]

>>> x = [i for i in range(5,-2,-1)]
>>> x
[5, 4, 3, 2, 1, 0, -1]

>>> x = [i for i in range(-5,-2,-1)]
>>> x
[]

>>> str = "0123456789"
>>> x = [str[i] for i in range(-1,-11,-1)]
>>> x
['9', '8', '7', '6', '5', '4', '3', '2', '1', '0']
```



### 参考

[python切片-截取-逆序截取](https://blog.csdn.net/HeyShHeyou/article/details/82665453)
[python逆序截取](<https://blog.csdn.net/win_turn/article/details/52998912>)



## zip
&#8195;&#8195;`zip()` 函数用于将可迭代的对象作为参数，如 **列表、元组、字符串**，将对象中对应的元素打包成一个个 **元组** ，然后返回由这些元组组成的对象。这比 Python 2 中直接返回列表节约了不少内存，我们可以使用 `list()` 转换来输出列表 。
&#8195;&#8195;如果各个迭代器的元素个数不一致，则返回列表 **长度与最短的对象相同** ，利用 `*` 号操作符，可以将元组解压为列表。

> zip 方法在 Python 2 和 Python 3 中的不同：在 Python 2.x zip() 返回的是一个列表，如果需要了解 Pyhton2 的应用，可以参考 [Python zip()](https://www.runoob.com/python/python-func-zip.html) 。



**zip 语法：**

> zip([iterable, ...])



**参数说明：**

> iterabl -- 一个或多个迭代器



**返回值**

> 返回一个对象。



&#8195;&#8195;示例如下：

``` python
>>> a = [1,2,3]
>>> b = [4,5,6]
>>> c = [4,5,6,7,8]
>>> zipped = zip(a,b)         # 返回一个对象
>>> zipped
<zip object at 0x103abc288>
>>> list(zipped)              # list() 转换为列表
[(1, 4), (2, 5), (3, 6)]
>>> list(zip(a,c))            # 元素个数与最短的列表一致
[(1, 4), (2, 5), (3, 6)]
 
>>> a1, a2 = zip(*zip(a,b))   # 与 zip 相反，zip(*) 可理解为解压，返回二维矩阵式
>>> list(a1)
[1, 2, 3]
>>> list(a2)
[4, 5, 6]
>>>
```



[Python zip() 函数](https://www.runoob.com/python3/python3-func-zip.html)



## enumerate

&#8195;&#8195;`enumerate()` 函数用于将一个可遍历的数据对象组合为一个索引序列，如 **列表、元组、字符串** ，同时列出数据和数据下标，一般用在 `for` 循环当中。



**enumerate 语法：**

> enumerate(sequence, [start=0])



**参数说明：**

> sequence -- 一个序列、迭代器或其他支持迭代对象。
> start -- 下标起始位置，可选。



**返回值：**

> 返回 enumerate(枚举) 对象。



&#8195;&#8195;示例如下：

```python
>>>seasons = ['Spring', 'Summer', 'Fall', 'Winter']
>>>list(enumerate(seasons))
[(0, 'Spring'), (1, 'Summer'), (2, 'Fall'), (3, 'Winter')]

>>>list(enumerate(seasons, start=1))       # 小标从 1 开始
[(1, 'Spring'), (2, 'Summer'), (3, 'Fall'), (4, 'Winter')]
```

[Python3 enumerate() 函数](https://www.runoob.com/python3/python3-func-enumerate.html)


## set

&#8195;&#8195;`set()` 函数创建一个 **无序** 的 **不重复** 元素序列，无序是指每次执行后顺序都可能不同，可以对 **元组、字符串** 使用，用大括号 `{ } `或者 `set()` 函数创建集合，**创建一个空集合必须用 set() 而不是 { }**，因为 `{ }` 是用来创建一个空字典。还可进行关系测试，删除重复数据，计算交集、差集、并集等。



**set 语法：**

> parame = {value01,value02,...}
> set(value)



&#8195;&#8195;示例如下：

```python
>>>basket = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
>>> print(basket)                      # 输出集合，重复的元素被自动去掉
{'orange', 'banana', 'pear', 'apple'}
>>> 'orange' in basket                 # 快速判断元素是否在集合内
True
>>> 'crabgrass' in basket
False
 
>>> # 下面展示两个集合间的运算.
...
>>> a = set('abracadabra')
>>> b = set('alacazam')
>>> a                                  
{'a', 'r', 'b', 'c', 'd'}
>>> a - b                              # 集合a中包含而集合b中不包含的元素
{'r', 'd', 'b'}
>>> a | b                              # 集合a或b中包含的所有元素
{'a', 'c', 'r', 'd', 'b', 'm', 'z', 'l'}
>>> a & b                              # 集合a和b中都包含了的元素
{'a', 'c'}
>>> a ^ b                              # 不同时包含于a和b的元素
{'r', 'd', 'b', 'm', 'z', 'l'}
```



## dict

&#8195;&#8195;列表是有序的对象集合，字典是 **无序** 的对象集合，两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。字典是一种映射类型，用 `{ }` 标识，它是一个无序的 `键(key) : 值(value)` 对集合。



**语法：**

> d = {key1 : value1, key2 : value2 }
>
> or
>
> dict(kwarg)
> dict(mapping, kwarg)
> dict(iterable, kwarg)



**参数说明：**

>  kwargs -- 关键字
> mapping -- 元素的容器。
> iterable -- 可迭代对象。



**实例：**

```python
>>>dict()                        # 创建空字典
{}
>>> dict(a='a', b='b', t='t')     # 传入关键字
{'a': 'a', 'b': 'b', 't': 't'}
>>> dict(zip(['one', 'two', 'three'], [1, 2, 3]))   # 映射函数方式来构造字典
{'three': 3, 'two': 2, 'one': 1} 
>>> dict([('one', 1), ('two', 2), ('three', 3)])    # 可迭代对象方式来构造字典
{'three': 3, 'two': 2, 'one': 1}
>>>
```



**有两点需要注意：**

> 1. 键必须是唯一的，但值不必，键(key) 必须使用不可变对象，值可以取任何数据类型，如字符串、数字、元组，不可以使用列表。
>
> 2. 不允许同一个键出现两次。创建时如果同一个键被赋值两次，后一个值会被记住。



### items


&#8195;&#8195;`items()` 方法以列表返回可遍历的 (键, 值) 元组数组。

**语法：**

> dict.items()
>
> 无参数，返回可遍历的 (键, 值) 元组数组。



**实例：**

```python
#!/usr/bin/python3
 
>>> dict = {'Name': 'Runoob', 'Age': 7}
>>> dict.items()
dict_items([('Name', 'Runoob'), ('Age', 7)])
>>> list(dict.items())
[('Name', 'Runoob'), ('Age', 7)]
```



### keys

&#8195;&#8195;Python3 字典 `keys()` 方法返回一个可迭代对象，可以使用 list() 来转换为列表。Python2.x 是直接返回列表。

**语法：**

> dict.keys()
>
> 无参数，返回一个迭代器。



**实例：**

```python
>>> dict = {'Name': 'Runoob', 'Age': 7}
>>> dict.keys()
dict_keys(['Name', 'Age'])
>>> list(dict.keys())             # 转换为列表
['Name', 'Age']
>>> 
```



获取字典的键值：

```python
if __name__ == '__main__':
    graph = {'A': {'B': 1, 'C': 12},
             'B': {'C': 9, 'D': 3},
             'C': {'E': 5},
             'D': {'E': 13, 'F': 15},
             'E': {'F': 4},
             'F': {}
             }
    for nodes in graph.keys():
        for w in graph[nodes].items():
            print(w)
            
# ('B', 1)
# ('C', 12)
# ('C', 9)
# ('D', 3)
# ('E', 5)
# ('E', 13)
# ('F', 15)
# ('F', 4)
```



## 元素操作

**对于 List：**

```python
classmates = ['Michael', 'Bob', 'Tracy']
classmates.append('Adam')    # 添加在末尾，没有 add()方法
classmates.insert(1, 'Jack') # 在指定位置添加
classmates.pop(1)            # 在知道位置删除，参数是索引
del classmate[1]             # 删除第二个元素
classmates.remove('Bob')     # 参数是元素，删除第一个与 Bob 值匹配的元素，之后又相同元素不会删除
```



**对于 dict：**

```python
d = {'a': 'A', 'b': 'B'}
del d['a']
d.pop('a')    # 参数是 key，没有 remove() 方法
d['c'] = 'C';   # 插入直接赋值即可
```



**对于set：**

```python
s = {1,2,3}     # set 对象的创建也可以是 s= set (iterable)
s.add(8)        # 添加 8 到末尾，没有 append() 方法
s.remove(8)     # 删除最后一个元素
```



**对于 tuple：**

&#8195;&#8195;由于 tuple 一旦初始化就不能修改，所以不能插入和删除



[Python可迭代对象中的添加和删除（add，append,pop,remove,insert）](https://blog.csdn.net/handsome_gay/article/details/52740055)




## None 和 if not

&#8195;&#8195;Python 中没有 Null ，取而代之的是 None，None 是一种特殊类型的对象，和其他对象差不多，只不过数据类型是 NoneType，值为 False 。

&#8195;&#8195;进行逻辑判断（比如 if）时，Python 当中等于 False 的值并不只有 False 一个，它也有一套规则。对于基本类型来说，基本上每个类型都存在一个值会被判定为 False，大致是如下：

> 1. 布尔型，False 表示False，其他为 True。
>
> 2. 整数和浮点数，0 表示 False，其他为True。
>
> 3. 字符串和类字符串类型（包括 bytes 和 unicode），空字符串表示 False，其他为 True。
>
> 4. 序列类型（包括 tuple、list、dict、set 等），空表示 False，非空表示 True。
> 
> 5. None 永远表示 False。

&#8195;&#8195;自定义类型则服从下面的规则：
> 1. 如果定义了 `__nonzero__()` 方法，会调用这个方法，并按照返回值判断这个对象等价于 True 还是 False。
> 2. 如果没有定义`__nonzero__()`方法但定义了 `__len__` 方法，会调用 `__len__` 方法，当返回 0 时为 False，否则为 True（这样就跟内置类型为空时对应 False 相同了）。
> 3. 如果都没有定义，所有的对象都是 True，只有 None 对应 False。




&#8195;&#8195;布尔非 `not` 是一个逻辑运算符，如果 x 为 True 则返回 False，如果 x 为 False 则返回 True。所以当 x 为 `None`、`False`、`空字符串""`、`0`、`空列表[]`、`空字典{}`、`空元组()`、`空集set()` 这些时，`if x` 为假，`if not x` 为真：

```python
>>> x = None
>>> not x
True
>>> x = False
>>> not x
True
>>> x = ""
>>> not x
True
>>> x = 0
>>> not x
True
>>> x = []
>>> not x
True
>>> x = {}
>>> not x
True
>>> x = ()
>>> not x
True

>>> x is not None
True
```

&#8195;&#8195;最后一行可以看出 `is not None` 只判断不为 None ，即使空元素也是真，但是 `not` 会判断不为 None 且不为 空。



[python里None 表示False吗？ （我是新手） - 灵剑的回答 - 知乎](https://www.zhihu.com/question/48707732/answer/112233903)



## 运算符

### 常规运算符

&#8195;&#8195;**算术运算符** 很常见，有 `+`、`-`、`*`、`%`，然后 `/` 是除以，`//` 是整除，`**` 是幂次方。

&#8195;&#8195;**比较（关系）运算符**很常见，有 `==`、`!=`、`>`、`<`、`>=`、`<=`。


&#8195;&#8195;**赋值运算符**很常见，有 `=`、`+=`、`-=`、`*=`、`/=`、`%=`、`**=`、`//=`。

&#8195;&#8195;**逻辑运算符**很常见，有 布尔与`and`、布尔或`or`、布尔非`not`。

&#8195;&#8195;**成员运算符**，`in` 表示如果在指定的序列中找到值返回 `True`，否则返回 `False`。`not in`正好相反。

&#8195;&#8195;**身份运算符**，`is` 是判断两个标识符是不是引用自一个对象，也就是如果两个对象的内存地址相同则返回 `True`，否则返回 `False`。`is not`正好相反。

&#8195;&#8195;**成员运算符**，如果在指定的序列中找到值返回 `True`，否则返回 `False`。`not in`正好相反。




### 位运算符

&#8195;&#8195;位运算符是把数字看作二进制来进行计算的：

| 运算符 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| &      | **按位与**：如果两个相应位都为 1 则该位的结果为 1 ，否则为 0 。 |
| \|     | **按位或**：只要对应的二个二进位有一个为 1 时，结果位就为 1 。 |
| ^      | **按位异或**：当两对应的二进位相异时，结果为 1 。            |
| ~      | **按位取反**：对数据的每个二进制位取反，即把 1 变为 0 把 0 变为1 。~x 就是 -x-1 |
| <<     | **左移运算符**：运算数的各二进位全部左移若干位，高位丢弃，低位补 0 。 |
| >>     | **右移运算符**：运算数的各二进位全部右移若干位。             |

&#8195;&#8195;按位取反 `~` 涉及到原码、反码和补码的转换：
> 1. 正数的原、反、补码都是它本身。
> 2. 负数的原码最高位是 1 开头；反码是最高符号位不变，其余位在原码的基础上取反；补码是在反码的基础上 +1 即可得到。



&#8195;&#8195;我们知道二进制数在内存中是以补码的形式存储的，所以一个十进制数按位取反的过程如下：

> a = 60   ~a = -61
> 二进制：    0011 1100
> 补码：        0011 1100   # 内存中就是这样存储
> 按位取反：1100 0011   
>
> **得到了按位取反后内存中的数，但现在还是补码形式，下面转成原码**
>
> 按位取反：1011 1100
> 末尾加一：1011 1101
> 转十进制：-(32+16+8+4+1) = -61



### is 和 ==

&#8195;&#8195;Python 中对象包含三个基本要素：id（身份标识）、type（数据类型）、value（值）。

&#8195;&#8195;`is` 是身份运算符，身份运算符用于比较两个对象的 **存储单元**，判断两个标识符是不是引用自一个对象，也就是比较内存地址（id）是否相同。`x is y` 就是 `id(x) == id(y) ` , 如果引用的是同一个对象则返回 `True`，否则返回 `False` 。`id()` 用于获取对象内存地址。

&#8195;&#8195;`==` 是比较运算符，用来判断两个对象的 value（值）是否相等。

```python
>>> a = 123
>>> b = 123
>>> a is b
False
>>> a == b
True

>>> a = ["1", "2", "3"]
>>> b = ["1", "2", "3"]
>>> a is b
False
>>> a == b
True
```



&#8195;&#8195;这里有个细节，如果把代码写在 `.py` 文件中运行，或者在 IDE 中运行，结果是不一样的：

```python
if __name__ == '__main__':
    a = 123
    b = 123
    print(a is b)
    
# True
```

&#8195;&#8195;这是 idle 里执行语句和执行整个 Python 文件的区别。简单说，idle 里执行是一条语句一条语句执行的，你可以理解为就是一个 loop，里面不断调用 exec 函数。而 Python 文件的执行是要先经过编译的，Python 虽然是解释型语言，但是也有编译这个步骤。所以 a 和 b 指向的对象是同一个，可能是因为 Python 解释器在编译的时候进行了优化，让它们指向了同一个对象。而 idle 没有这个步骤，它肯定不可能预知未来，在你还没输入 b 的时候就进行优化 。


[Python解释器里的is和Pycharm里的is有什么区别？ - 松直的回答 - 知乎](https://www.zhihu.com/question/315850653/answer/622379099)



## for 循环原理

&#8195;&#8195;Python 中的 for 严格来说不是循环，是一种迭代，循环可以通过增加条件跳过不需要的元素，而迭代则只能一个一个的往后取数据。在程序设计中，通常会有 loop、iterate、traversal 和 recursion 等概念，他们各自的含义如下：

> 1. 循环（loop），在满足条件的情况下，重复执行同一段代码。比如 Python 中的 while 语句。
> 2. 迭代（iterate），按照某种顺序逐个访问迭代器中的每一项。比如 Python 中的 for 语句。
> 3. 递归（recursion），一个函数不断调用自身。比如常见的用递归或者迭代实现斐波纳契数列。
> 4. 遍历（traversal），按照一定的规则访问树形结构中的每个节点，而且每个节点都只访问一次。



&#8195;&#8195;很多语言中 for 与 while 都用于循环，而 Python 中严格地说只有 while 是循环， for 其实是迭代，它的结构是 `for ... in ...`，其在迭代时会产生迭代器，实际是将可迭代对象转换成迭代器，再重复调用 `next()` 方法实现的。例如：

```python
for x in [1, 2, 3, 4, 5]:
    pass
```
&#8195;&#8195;实际上完全等价于：
```python
it = iter([1, 2, 3, 4, 5])
while True:
    try:
        x = next(it)
    except StopIteration:
        break
```




&#8195;&#8195;这里提到两个概念：可迭代对象（Iterable）、迭代器（Iterator）。



### Iterator

**iterable（可迭代对象）**

&#8195;&#8195;一个可迭代对象要么定义了 `__iter__` 方法，用于返回一个迭代器；要么定义了 `__getitem__` 方法，可以按 index 索引的对象（并且能够在没有值时抛出一个 `IndexError` 异常），这是一种按下标迭代的方式。可迭代对象具有如下的特性：

> 1. 可以for循环： `for i in iterable` ；
> 2. 可以按 index 索引的对象，也就是定义了 `__getitem__` 方法，比如 `list`、`str` ；
> 3. 定义了 `__iter__` 方法。可以随意返回；
> 4. 可以调用 `iter(obj)` 的对象，并且返回一个 iterator 。



**iterator（迭代器对象）**

&#8195;&#8195;迭代器对象就是实现了迭代器协议的对象，迭代器协议如下：

> 1. 定义了 `__iter__` 方法，但是必须返回自身；
> 2. 定义了 `__next__` 方法，用来返回下一个值，并且当没有数据时，抛出 `StopIteration` 异常；
> 3. 可以保持当前的状态。



&#8195;&#8195;Iterable 的 `__iter__` 方法会返回一个 Iterator，Iterator 的 `__next__` 方法会返回下一个迭代对象，如果迭代结束则抛出 `StopIteration` 异常。可以得知 Iterator 对象要实现两个方法 `__iter__()` 和 `__next__()` ，可是 Iterable 已经实现了 `__iter__` 方法，Iterator 为什么还要实现 `__iter__` 方法，而且是返回自己呢？确实 Iterator 只需实现 `__next__()` 方法就可以了，但是 Iterator 自己也是一种 Iterable，所以也需要实现 Iterable 的接口，也就是 `__iter__`，这样在 `for` 当中两者就都可以使用了，Iterator 的 `__iter__` 只需要返回自己就行了。



**Iterable 变 Iterator**

&#8195;&#8195;str、list、dict等都 是 `iterable` 而不是 `iterator`，把 list、dict、str 等 Iterable 变成 Iterator 可以使用 `iter()` 函数（`iter()` 函数不同于 `__iter__` 方法）：

```python
from collections.abc import Iterable, Iterator
>>>list=[1,2,3,4]
>>>isinstance(list, Iterable)
True
isinstance(iter(list), Iterator)
False
>>> it = iter(list)    # 创建迭代器对象
>>> print (next(it))   # 输出迭代器的下一个元素
1
>>> print (next(it))
2
>>>
```



&#8195;&#8195; 一个类想要像 list 或 tuple 那样被用于 `for ... in` 循环，就必须实现一个 `__iter__()` 方法，该方法返回一个迭代对象，然后，Python 的 for 循环就会不断调用该迭代对象的 `next()` 方法拿到循环的下一个值，直到遇到 StopIteration 错误时退出循环。以斐波那契数列为例，写一个可用于 for 循环的 Fib 类：

```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1 # 初始化两个计数器a，b

    def __iter__(self):
        return self # 实例本身就是迭代对象，故返回自己

    def next(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 10: # 退出循环的条件
            raise StopIteration();
        return self.a # 返回下一个值
```

效果：

```python
>>> for n in Fib():
...     print n
...
1
1
2
3
5
```



&#8195;&#8195;Python 的 for 循环本质上就是通过不断调用 `next()` 函数实现的。当遍历一个迭代器的时候，它会修改内部状态，导致你只能获取下一个元素，不能通过迭代器访问后面一个元素；也就是说当你通过迭代器访问了一个元素以后，在当前循环中不能后退继续访问该元素了，除非你重新生产迭代器对象进行遍历。



[对 Python 迭代的深入研究](<http://kuanghy.github.io/2016/05/18/python-iteration>)

[python的迭代器为什么一定要实现__iter__方法？](<https://www.zhihu.com/question/44015086>)



### Generators

&#8195;&#8195;如果我们创建一个包含 100 万个元素的列表，但是仅仅需要访问前面几个元素，这样不仅占用很大的存储空间，后面绝大多数元素占用的空间都白白浪费了。如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的 list，从而节省大量的空间。在 Python 中，这种一边循环一边计算的机制，称为生成器（generator）。

**第一种定义方法**
&#8195;&#8195;第一种方法很简单，只要把一个列表生成式的 `[]` 改成 `()`，就创建了一个 generator，可以不断调用 `next()` 函数获得 generator 的下一个返回值，直到抛出 `StopIteration` 错误：

```python
>>> L = [x * x for x in range(10)]
>>> L
>>> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
>>> g = (x * x for x in range(10))
>>> g
>>> <generator object <genexpr> at 0x1022ef630>
>>> next(g)
0
>>> next(g)
1
...
>>> next(g)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

&#8195;&#8195;我们创建了一个 generator 后，基本上不会手动调用 `next()`，而是通过 `for` 循环来迭代它，并且不需要关心 `StopIteration` 的错误。



**第二种定义方法**

&#8195;&#8195;如果推算的算法比较复杂，用类似列表生成式的 for 循环无法实现的时候，还可以用函数来实现。比如著名的斐波拉契数列（Fibonacci）用列表生成式是很难实现的，用函数却很容易：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'

# 1 1 2 3 5 8
# 'done'
```



&#8195;&#8195;实现 generator 只需要一个简单的 `yield`。`yield` 是生成器实现 `__next__()` 方法的关键。它作为生成器执行的暂停恢复点，可以对 `yield` 表达式进行赋值，也可以将 `yield` 表达式的值返回。任何包含 `yield` 语句的函数被称为生成器：

```python
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        print(b)
        a, b = b, a + b
        n = n + 1
    return 'done'

>>> f = fib(6)
>>> f
<generator object fib at 0x104feaaa0>
```

&#8195;&#8195;generator 和函数的执行流程不一样。函数是顺序执行，遇到 return 语句或者最后一行函数语句就返回。而变成 generator 的函数，**在每次调用 next() 的时候执行，遇到 yield 语句返回，再次执行时从上次返回的 yield 语句处继续执行。**循环调用 generator 时，想拿到 generator 的 return 语句的返回值，必须捕获 StopIteration 错误，返回值包含在 StopIteration 的 value 中：

```python
>>> g = fib(6)
>>> while True:
...     try:
...         x = next(g)
...         print('g:', x)
...     except StopIteration as e:
...         print('Generator return value:', e.value)
...         break
...
g: 1
g: 1
g: 2
g: 3
g: 5
g: 8
Generator return value: done
```



&#8195;&#8195;迭代器是一个对象，而生成器是一个函数，函数中每次使用 yield 产生一个值，函数就返回该值，然后停止执行，等待被激活，被激活后继续在原来的位置执行。



[生成器 · 廖雪峰的Python3.x教程 · 看云](<https://www.kancloud.cn/smilesb101/python3_x/296108>)



### 对 range 迭代

&#8195;&#8195;在 C 语言中 for 是依靠变量自增来实现循环的，循环体内对变量的改变会影响循环过程，但是在 Python 中，循环体内变量的改变并不影响循环过程，for 循环是靠 `range()` 决定的：

```c
#include <stdio.h>

int main(){
	int i;
    for(i = 0; i< 10; i++){		
   	    i ++;
	    printf("%d",i);
    }
}

# 1 3 5 7 9
```



```python
if __name__ == '__main__':
    for i in range(0, 10):
        i += 1
        print(i)
        
# 1 2 3 4 5 6 7 8 9 10
```

&#8195;&#8195;while 循环可以通过增加条件跳过不需要的元素，而 for 迭代则只能一个一个的往后取数据，上面代码中，无论对 `i` 做什么操作，接下来的循环始终指向 `range(0, 10)` 中的下一个元素。但是，如果对列表做迭代，列表发生变化时，是可能影响到循环过程的。



### 对列表迭代

**迭代过程不变**

&#8195;&#8195;下面代码在迭代过程中对列表进行修改，但是并没有使循环过程发生变化，列表虽然从 `f[1] = [44, 55, 66]` 变成了 `f[1] = [1, 2, 3]` ，但迭代依然过程依然是 `[44, 55, 66]` ，这里我觉得是因为迭代开始时，就已经指向了 `[44, 55, 66]`  这个对象的内存地址，接下来每次迭代 `__next__` 方法都会返回后面的元素，虽然过程中 `f[1]` 指向了新的地址，但原地址中元素并没有变，所以迭代器依次读取元素也不会变。

```python
if __name__ == '__main__':   
    f = [[1, 2, 3], [7, 8, 9]]
    g = f[1]
    print("befor f:%s\nbefore g:%s" % (f, g))
    f[1] = [44, 55, 66]
    print("after f:{}\nafter g:{}".format(f, g))
    for i in f[1]:
        print("i:", i)
        if i == 44:
            f[1] = f[0]
        print("f:", f)
        
# befor f:[[1, 2, 3], [7, 8, 9]]
# before g:[7, 8, 9]
# after f:[[1, 2, 3], [44, 55, 66]]
# after g:[7, 8, 9]
# i: 44
# f: [[1, 2, 3], [1, 2, 3]]
# i: 55
# f: [[1, 2, 3], [1, 2, 3]]
# i: 66
# f: [[1, 2, 3], [1, 2, 3]]
```

&#8195;&#8195;当然如果有 `f[2]` ，我们在 `f[1]` 迭代过程中让 `f[2]` 指向了新的列表，那迭代到 `f[2]` 时候就会是新的列表，如下 `i` 会打印出什么结果：

```python
if __name__ == '__main__':   
    f = [[1, 2, 3], [7, 8, 9], [6, 6, 6]]
    for i in f:
        for j in i:
            print("j:", j)
            if j == 7:
                f[2] = f[0]
        print("f:", f)
```



&#8195;&#8195;不建议在 for 迭代过程中对列表进行修改，比如对列表进行了删除操作，那列表实长级就会缩短，迭代器就可能产生越界，然后报错。



**迭代过程变化**

&#8195;&#8195;下面代码的循环过程就发生了改变

```python
if __name__ == '__main__':   
    f = [[1, 2, 3], [7, 8, 9]]
    g = f[1]
    print("befor f:%s\nbefore g:%s" % (f, g))
    f[1] = [44, 55, 66]
    print("after f:{}\nafter g:{}".format(f, g))    
    for i in f[1]:
        print("i:", i)
        if i == 44:
            f[1][1] = f[0]
        print("f:", f)
        
# befor f:[[1, 2, 3], [7, 8, 9]]
# before g:[7, 8, 9]
# after f:[[1, 2, 3], [44, 55, 66]]
# after g:[7, 8, 9]
# i: 44
# f: [[1, 2, 3], [44, [1, 2, 3], 66]]
# i: [1, 2, 3]
# f: [[1, 2, 3], [44, [1, 2, 3], 66]]
# i: 66
# f: [[1, 2, 3], [44, [1, 2, 3], 66]]
```

&#8195;&#8195;这里发现迭代过程发生了改变，为什么修改 `f[1] ` 不会影响迭代过程，但是修改 `f[1][1]` 会对迭代过程产生了影响？我觉得原因如下： `f[1] ` 的修改是让 `f[1] ` 指向了一个新的地址，原地址元素不变，迭代器一开始就是作用域原地址的对象，所以只要原地址中元素不变，迭代过程就不变；但是修改 `f[1][1]` 却是直接修改了原内存地址中的元素，（这部分不懂的话需要了解一下 [可变对象和不可变对象](<https://www.gokuweb.com/code/e58ae62f.html>) ），原地址元素从 `[44, 55, 66]`  变成了 `[44, [1, 2, 3], 66]]` ，所以迭代器也跟着发生变化。

[Why can’t you modify lists through "for in" loops in Python?]([https://www.quora.com/Why-can%E2%80%99t-you-modify-lists-through-for-in-loops-in-Python](https://www.quora.com/Why-can't-you-modify-lists-through-for-in-loops-in-Python))



## join 和 +

&#8195;&#8195;`join()` 方法用于将序列中的元素以指定的字符连接生成一个新的字符串。

**join() 语法：**

> str.join(sequence)



**参数说明：**

> sequence -- 要连接的元素序列



**返回值**

> 返回通过指定字符连接序列中元素后生成的新字符串



&#8195;&#8195;应避免在循环中用 `+` 和 `+=` 操作符来累加字符串，由于字符串是不可变对象， 这样做会创建不必要的临时对象，并且导致二次方而不是线性的运行时间。作为替代方案，你可以将每个子串加入列表，然后在循环结束后用 `.join` 连接列表（也可以将每个子串写入一个 `cStringIO.StringIO` 缓存中）。

&#8195;&#8195;字符串操作时没有 `append` 的，在用 `+ ` 连接字符串时，结果会生成新的对象，用 `join` 时结果只是将原列表中的元素拼接起来，所以 `join` 效率比较



[Python风格规范](https://zh-google-styleguide.readthedocs.io/en/latest/google-python-styleguide/python_style_rules/#id12)
