---
title: 细说二分查找（Binary Search）
categories:
  - 算法
tags:
  - bs
abbrlink: bc66874b
date: 2019-05-30 13:39:14
---

&#8195;&#8195;据说十个二分九个错，如果单靠背的话，的确稍有变形就容易出错，二分查找的前提是 **非递减序列** ，本质上是不断折半缩小区间，难点在于边界的控制，避免陷入死循环和漏查的情况。
<!-- more -->
&#8195;&#8195;二分查找应用的场景有很多，比如查找非递减序列中是否存在 val、返回非递减序列中第一个等于 val 的元素位置、返回非递减序列中 第一个大于 val 的元素位置等，代码上稍有不同。

## 二分查找函数
&#8195;&#8195;C++ 中有三个二分查找的函数，我们用 python 改写一下，实现相同的功能。

### lower_bound
&#8195;&#8195;`lower_bound(first, last, val)`，在 **非递减序列** 的 **左闭右开** 区间 `[first, last)` 中进行二分查找，返回 **大于或等于 val 的 <font color="red">第一个</font>** 元素位置。如果所有元素都小于 val，则返回 last 的位置，此时 last 的位置是越界的。
```python
def lower_bound(array, first, last, value):
    while first < last:
        #这里不用 mid = (first + last)//2 是为了防溢出
        mid = first + (last - first)//2  #虽然 python 不会溢出
        if array[mid] < value:
            first = mid + 1
        else:
            last = mid
    return first # last也行，因为[first, last)为空的时候它们重合
```

> &#8195;&#8195;如果想找 **小于或等于 val 的 <font color="red">最后一个</font>** 元素位置，可以使用 `upper_bound() - 1` 得到。比如可以用 `upper_bound(0, 9, 4)`返回 "0 1 2 3 3 4 4 4 8" 中小于等于 4 的最后一个元素（即第三个 4）。

### upper_bound
&#8195;&#8195;`upper_bound(first, last, val)`，在 **非递减序列** 的 **左闭右开** 区间 `[first, last)` 中进行二分查找，返回 **大于 val 的<font color="red">第一个</font>** 元素位置。如果 val 大于数组中全部元素，则返回 last 的位置，此时 last 的位置是越界的。跟上面函数的区别仅仅是 `if array[mid] <= value`。
```python
def upper_bound(array, first, last, value):
    while first < last:
        mid = first + (last - first)//2  # 防溢出
        if array[mid] <= value:
            first = mid + 1
        else:
            last = mid
    return first # last也行，因为[first, last)为空的时候它们重合
```

> &#8195;&#8195;如果想找 **小于 val 的 <font color="red">最后一个</font>** 元素位置，可以使用 `lower_bound() -1` 得到。比如可以用 `lower_bound(0, 9, 4)`返回 "0 1 2 3 3 4 4 4 8" 中小于 4 的最后一个元素（即第二个 3）。

### binary_search
&#8195;&#8195;`binary_search(first, last, val)`，在 **非递减序列** 的 **左闭右开** 区间 `[first, last)` 中查找 **是否存在 val** ，不存在则返回 -1 。
```python
def binary_search(self, array, first, last, value):
    while first < last:
        mid = first + (last - first)//2
        if array[mid] < value:
            first = mid + 1
            print(first, last)
        elif array[mid] > value:
            last = mid
            print(first, last)
        else:
            return mid
    return -1
```

如下图：
![bsfunction](/imgs/bsfunction.png)

## 边界问题
&#8195;&#8195;二分查找的原理是利用区间内值有序的特点，不断让可行区间减半，最终可行区间长度减到 1 得到答案，要保证二分查找能得到正确答案，并且不会死循环，要保证两个条件:
> 1. 解一直在可行区间里。
> 2. 每次判断后可行区间都会缩小（特别是左右端点相距为 1 的时候）。

&#8195;&#8195;第一点不满足会产生 **漏查**，比如在左闭右开区间 `while first < last` 查找 val，假如 val 元素的位置是 3 而区间又正好收敛到 `[3, 3)` 时候，由于不符合循环条件就导致了漏查。
&#8195;&#8195;第二点不满足会产生 **死循环**，在查找区间长度大于 2 的时候不会出问题，但是当可行区间左右端点相距为 1 的时候，比如 `[2, 3)` ，如果使用 `first = mid`，那向右收敛的过程就会陷入 `[2, 3)` 的死循环中。

### 循环不变量
&#8195;&#8195;怎样缩小区间才不会出错？首先需要理解 while 循环里的循环不变量（loop invariants），也就是代码跑到 while 里面时一定成立的条件。循环不变量（loop invariants）主要用来帮助理解算法的正确性，形式上很类似与数学归纳法，对于循环不变量，必须证明它的三个性质：
> 1. 初始化：它在循环的第一轮迭代开始之前，应该是正确的。
> 2. 保持：如果在循环的某一次迭代开始之前它是正确的，那么，在下一次迭代开始之前，它也应该保持正确。
> 3. 终止：循环能够终止，并且可以得到期望的结果。

对照 `lower_bound` 函数分析一下：
> 1. 搜索范围 `[first, last)` 不为空，即 first < last ；
> 2. 搜索范围 `[first, last)` 左侧，即 `[first0, first)` 内所有元素(若存在)，都小于 value，其中 first0 是 first 的初始值；
> 2. 搜索范围 `[first, last)` 右侧，即 `[last, last0)` 内所有元素(若存在)，都大于等于 value，其中 last0 是 last 的初始值。

再看一次 `lower_bound()` 代码：
```python
def lower_bound(array, first, last, value):
    while first < last: # 搜索区间[first, last)不为空
        mid = first + (last - first)//2  # 防溢出
        if array[mid] < value:
            first = mid + 1
        else:
            last = mid
    return first # last也行，因为此时重合
```
&#8195;&#8195;为什么 `first = mid + 1` 而 `last = mid` 呢？这实际上是由 `while first < last` 决定的，这是一个 **左闭右开** 的区间，所以 first 和 last 变动时候也要按照左闭右开为准。我们把 `[first, last)` 拆分成 `[first, mid) mid [mid + 1, last)`：
> 1. 当 `array[mid] < value` 时，val 一定在 `[mid+1, last)` 区间内，所以 `if array[mid] < value: first = mid + 1`，虽然说 `[mid+1, last)` 和 `(mid, last)` 是等价的，但为了与原始区间一致我们只能用左闭右开的形式；
> 2. 当 `array[mid] >= value` 时，val 一定在 `[first, mid-1]` 区间内，同样为了与原始区间一致，我们只能用左闭右开的形式 `[first, mid)`，所以 `if array[mid] >= value: last = mid` ，假如用 `last = mid - 1` 和 `first = mid + 1`，那区间变化无非就是 `[first, mid-1)` 或者 `[mid+1, last)`，左闭右开 `[first, mid-1)` 中的 mid -1 会被漏查，比如 `[2, 2)`，此时 不满足循环条件，该位置的元素就不会进行查找。

### 区间缩减
&#8195;&#8195;二分查找中让人头大的一般是下面三点：
> 1. `while first < last` 还是 `while first <= last` ？
> 2. `if array[mid] < value` 还是 `if array[mid] <= value` ？
> 3. `first = mid + 1` 还是 `first = mid` ？ `last = mid + 1` 还是 `last = mid` ？ 

&#8195;&#8195;第一三点是造成 **死循环** 和 **漏找** 的主要原因，本节主要将这两点。第二点一般用于处理重复元素，比如返回第一个大于等于 val 的元素还是返回第一个大于 val 的元素。我们看看在非递减序列中查找 val 的几种不同写法：

** 左闭右开 while first < last**
```python
class Solution:
    # 正确
    def search_val1(self, array, first, last, value):
        while first < last:
            mid = first + (last - first)//2
            if array[mid] < value:
                first = mid + 1
            elif array[mid] > value:
                last = mid
            else:
                return mid
        return -1
    
    # 错误，死循环
    def search_val2(self, array, first, last, value):
        while first < last:
            mid = first + (last - first)//2
            if array[mid] < value:
                first = mid
            elif array[mid] > value:
                last = mid - 1
            else:
                return mid
        return -1
 
    # 错误，死循环
    def search_val3(self, array, first, last, value):
        while first < last:
            mid = first + (last - first)//2
            if array[mid] < value:
                first = mid
            elif array[mid] > value:
                last = mid
            else:
                return mid
        return -1 
 
    # 错误，漏找
    def search_val4(self, array, first, last, value):
        while first < last:
            mid = first + (last - first)//2
            if array[mid] < value:
                first = mid + 1
            elif array[mid] > value:
                last = mid - 1
            else:
                return mid
        return -1    

if __name__ == '__main__':
    S = Solution()
    test = [0, 1, 2, 3, 4, 4, 4, 4, 8]
    print("search_val1:", S.search_val1(test, 0, len(test), 3))

# search_val1: 3
```

我们看一些 testcase：

search_val1 区间按如下缩减：
> 1. while first < last
> 2. first = mid + 1 
> 3. last = mid

|  | search_val1(test, <br>0, len(test), -2) | search_val1(test, <br>0, len(test), 2.5) | search_val1(test, <br>0, len(test), 3) | search_val1(test, <br>0, len(test), 5) | search_val1(test, <br>0, len(test), 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 4) | [0, 4) | [0, 4) | [5, 9) | [5, 9) |
|第 2 次循环区间| [0, 2) | [3, 4) | [3, 4) | [8, 9) | [8, 9) |
|第 3 次循环区间| [0, 1) | [3, 3) |        | [8, 8) | [9, 9) |
|第 4 次循环区间| [0, 0) |        |        |        |        |
|     结果      |   -1   |   -1   |   3    |   -1   |   -1   |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |

&#8195;&#8195;search_val1 是正确的，原始区间 `[first, last)` 左闭右开，缩减后是 `[first, mid)` 和 `[mid + 1, last)`，无论向左还是向右缩减，只要找不到 val 最终就会走到 `first = last`，结束循环，不存在漏找和死循环。

search_val2 区间按如下缩减：
> 1. while first < last
> 2. first = mid 
> 3. last = mid - 1

|  | search_val2(test, <br>0, len(test), -2) | search_val2(test, <br>0, len(test), 0) | search_val2(test, <br>0, len(test), 3) | search_val2(test, <br>0, len(test), 5) | search_val2(test, <br>0, len(test), 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 3) | [0, 3) | [0, 3) | [4, 9) | [4, 9) |
|第 2 次循环区间| [0, 0) | [0, 0) | [1, 3) | [6, 9) | [6, 9) |
|第 3 次循环区间|        |        | [2, 3) | [7, 9) | [7, 9) |
|第 4 次循环区间|        |        | [2, 3) | [7, 7) | [8, 9) |
|     结果      |   -1   |-1(漏找)| 死循环 |   -1   | 死循环 |
|   期望结果    |   -1   |    0   |   3    |   -1   |   -1   |

&#8195;&#8195;search_val2 是错误的，原始区间 `[first, last)` 左闭右开，缩减后是 `[first, mid - 1)` 和 `[mid, last)`，第一个问题就是 mid - 1 这个位置会漏找，如区间为 `[0, 0)` 的时候不会进入循环体，产生了漏找；第二个问题就是死循环，产生死循环的原因是：1. 用 `first = mid` 向右缩减，2. 我们的 mid 是向下取整，所以当区间长度为 1 时候，比如 `[2, 3)`，mid 向下取整是 2，`first = mid` 还是 2，陷入死循环。只要长度为 1 且向右缩减，就会产生死循环，表中对 0 的查找始终向左缩减所以不会死循环，对 5 的查找不产生长度为 1 的 区间所以也不会死循环。

search_val3 区间按如下缩减：
> 1. while first < last
> 2. first = mid 
> 3. last = mid

|  | search_val3(test, <br>0, len(test), -2) | search_val3(test, <br>0, len(test), 2.5) | search_val3(test, <br>0, len(test), 3) | search_val3(test, <br>0, len(test), 5) | search_val3(test, <br>0, len(test), 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 4) | [0, 4) | [0, 4) | [4, 9) | [4, 9) |
|第 2 次循环区间| [0, 2) | [2, 4) | [2, 4) | [6, 9) | [6, 9) |
|第 3 次循环区间| [0, 1) | [2, 3) |        | [7, 9) | [7, 9) |
|第 4 次循环区间| [0, 0) | [2, 3) |        | [7, 8) | [8, 9) |
|     结果      |   -1   | 死循环 |   3    | 死循环 | 死循环 |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |

&#8195;&#8195;search_val3 是错误的，原始区间 `[first, last)` 左闭右开，缩减后是 `[first, mid)` 和 `[mid, last)`，区间有重合不存在漏找，但是会产生死循环，产生死循环的原因同上，向下取整的 mid 遇到 `first = mid`，当区间长度为 1 时候，比如 `[2, 3)`，mid 和 first 始终为 2 ，陷入死循环。只要长度为 1 且向右缩减，就会产生死循环，向左缩减没事，因为 mid 向下取整，比如 `[0, 1)` 会变成 `[0, 0)`。

search_val4 区间按如下缩减：
> 1. while first < last
> 2. first = mid + 1 
> 3. last = mid - 1

|  | search_val4(test, <br>0, len(test), -2) | search_val4(test, <br>0, len(test), 2.5) | search_val4(test, <br>0, len(test), 3) | search_val4(test, <br>0, len(test), 5) | search_val4(test, <br>0, len(test), 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 3) | [0, 3) | [0, 3) | [5, 9) | [5, 9) |
|第 2 次循环区间| [0, 0) | [2, 3) | [2, 3) | [8, 9) | [8, 9) |
|第 3 次循环区间|        | [3, 3) | [3, 3) | [8, 7) | [9, 9) |
|第 4 次循环区间|        |        |        |        |        |
|     结果      |   -1   |   -1   |-1(漏找)|   -1   |   -1   |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |

&#8195;&#8195;search_val4 是错误的，原始区间 `[first, last)` 左闭右开，缩减后是 `[first, mid - 1)` 和 `[mid + 1, last)`，不存在死循环但是会漏找，左闭右开的区间中 mid - 1 这个位置会漏找，如区间为 `[3, 3)` 的时候不会进入循环体，产生了漏找。

 
** 左闭右闭 while first <= last**
```python
class Solution:
    # 错误，死循环
    def search_val5(self, array, first, last, value):
        while first <= last:
            mid = first + (last - first) // 2
            if array[mid] < value:
                first = mid + 1
                print(first, last)
            elif array[mid] > value:
                last = mid
                print(first, last)
            else:
                return mid
        return -1

    # 错误，死循环
    def search_val6(self, array, first, last, value):
        while first <= last:
            mid = first + (last - first) // 2
            if array[mid] < value:
                first = mid
                print(first, last)
            elif array[mid] > value:
                last = mid - 1
                print(first, last)
            else:
                return mid
        return -1

    # 错误，死循环
    def search_val7(self, array, first, last, value):
        while first <= last:
            mid = first + (last - first) // 2
            if array[mid] < value:
                first = mid
                print(first, last)
            elif array[mid] > value:
                last = mid
                print(first, last)
            else:
                return mid
        return -1

    # 正确
    def search_val8(self, array, first, last, value):
        while first <= last:
            mid = first + (last - first) // 2
            if array[mid] < value:
                first = mid + 1
                print(first, last)
            elif array[mid] > value:
                last = mid - 1
                print(first, last)
            else:
                return mid
        return -1

if __name__ == '__main__':
    S = Solution()
    test = [0, 1, 2, 3, 4, 4, 4, 4, 8]
    print("search_val5:", S.search_val5(test, 0, len(test) - 1, 3))
```

search_val5 区间按如下缩减：
> 1. while first <= last
> 2. first = mid + 1 
> 3. last = mid

|  | search_val5(test, <br>0, len(test)-1, -2) | search_val5(test, <br>0, len(test)-1, 2.5) | search_val5(test, <br>0, len(test)-1, 3) | search_val5(test, <br>0, len(test)-1, 5) | search_val5(test, <br>0, len(test)-1, 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 4] | [0, 4] | [0, 4] | [5, 8] | [5, 8] |
|第 2 次循环区间| [0, 2] | [3, 4] | [3, 4] | [7, 8] | [7, 8] |
|第 3 次循环区间| [0, 1] | [3, 3] |        | [8, 8] | [8, 8] |
|第 4 次循环区间| [0, 0] |        |        | [8, 8] | [9, 8] |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |
|     结果      | 死循环 | 死循环 |   3    | 死循环 |   -1   |

&#8195;&#8195;search_val5 是错误的，原始区间 `[first, last]` 左闭右闭，缩减后是 `[first, mid]` 和 `[mid + 1, last]`，不会漏找但是会死循环，产生死循环的原因是：1. 用 `last = mid` 向左缩减，2. 当 `first = last` 时候还会循环。所以当 `first = last` 时候，比如 `[3, 3]`，last 和 mid 始终是 3 ，陷入死循环。只要走到 `first = last` 这一步，就会产生死循环。

search_val6 区间按如下缩减：
> 1. while first <= last
> 2. first = mid 
> 3. last = mid - 1

|  | search_val6(test, <br>0, len(test)-1, -2) | search_val6(test, <br>0, len(test)-1, 2.5) | search_val6(test, <br>0, len(test)-1, 3) | search_val6(test, <br>0, len(test)-1, 5) | search_val6(test, <br>0, len(test)-1, 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 3] | [0, 3] | [0, 3] | [4, 8] | [4, 8] |
|第 2 次循环区间| [0, 0] | [1, 3] | [1, 3] | [6, 8] | [6, 8] |
|第 3 次循环区间| [0, -1]| [2, 3] | [2, 3] | [7, 8] | [7, 8] |
|第 4 次循环区间|        | [2, 3] | [2, 3] | [7, 8] | [7, 8] |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |
|     结果      |   -1   | 死循环 | 死循环 | 死循环 | 死循环 |

&#8195;&#8195;search_val6 是错误的，原始区间 `[first, last]` 左闭右闭，缩减后是 `[first, mid - 1]` 和 `[mid, last]`，区间有重合不会漏找但是会死循环，产生死循环的原因同 search_val2 类似，向下取整的 mid 遇到 `first = mid`，当区间长度为 1 时候，比如 `[2, 3]`，mid 和 first 始终为 2 ，陷入死循环。只要长度为 1 且向右缩减，就会产生死循环，向左缩减没事，因为 `last = mid - 1` 会一直走到 last < first 结束，比如 `[0, 0]` 会变成 `[0, -1]` 结束循环。

search_val7 区间按如下缩减：
> 1. while first <= last
> 2. first = mid
> 3. last = mid

|  | search_val7(test, <br>0, len(test)-1, -2) | search_val7(test, <br>0, len(test)-1, 2.5) | search_val7(test, <br>0, len(test)-1, 3) | search_val7(test, <br>0, len(test)-1, 5) | search_val7(test, <br>0, len(test)-1, 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 4] | [0, 4] | [0, 4] | [4, 8] | [4, 8] |
|第 2 次循环区间| [0, 2] | [2, 4] | [2, 4] | [6, 8] | [6, 8] |
|第 3 次循环区间| [0, 1] | [2, 3] |        | [7, 8] | [7, 8] |
|第 4 次循环区间| [0, 0] | [2, 3] |        | [7, 8] | [7, 8] |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |
|     结果      | 死循环 | 死循环 |   3    | 死循环 | 死循环 |

&#8195;&#8195;search_val7 是错误的，原始区间 `[first, last]` 左闭右闭，缩减后是 `[first, mid]` 和 `[mid, last]`，区间有重合不会漏找但是会死循环。在两种情况下会死循环：1. 当 `first = last` 时候不会结束循环，此时还没找到 val 的话就会陷入死循环，比如区间 `[0, 0]` 内没找到 val 的话，就会死循环。2. 当 `first + 1 = last` 时候，比如 `[2, 3]`，如果没有找到 val 就会死循环，只要长度为 1 且向右缩减，就会产生死循环，向左缩减就成了第一种情况，因为 mid 向下取整，比如 `[0, 1]` 会变成 `[0, 0]`。

search_val8 区间按如下缩减：
> 1. while first <= last
> 2. first = mid + 1 
> 3. last = mid - 1

|  | search_val8(test, <br>0, len(test)-1, -2) | search_val8(test, <br>0, len(test)-1, 2.5) | search_val8(test, <br>0, len(test)-1, 3) | search_val8(test, <br>0, len(test)-1, 5) | search_val8(test, <br>0, len(test)-1, 9) |
|-----|----|----|----|----|----|
|第 1 次循环区间| [0, 3] | [0, 3] | [0, 3] | [5, 8] | [5, 8] |
|第 2 次循环区间| [0, 0] | [2, 3] | [2, 3] | [7, 8] | [7, 8] |
|第 3 次循环区间| [0, -1]| [3, 3] | [3, 3] | [8, 8] | [8, 8] |
|第 4 次循环区间|        | [3, 2] |        | [8, 7] | [9, 8] |
|   期望结果    |   -1   |   -1   |   3    |   -1   |   -1   |
|     结果      |   -1   |   -1   |   3    |   -1   |   -1   |

&#8195;&#8195;search_val8 是正确的，原始区间 `[first, last]` 左闭右闭，缩减后是 `[first, mid + 1]` 和 `[mid - 1, last]`，向左缩减会走到 `[0, -1]` 结束，向右缩减会走到 `[9, 8]` 结束，不存在漏找和死循环。

&#8195;&#8195;综上其实只要注意两点就行：
> 1. 子区间要和原区间一直，要么都 左闭右开，要么都左闭右闭；
> 2. 子区间和原区间不要重合，不要遗漏。

如下：
first = 0
last = len(test)

|                                 | while first < last            | while first <= last - 1 |
| -- | -- | -- |
|原始区间                         |[first, last)                  |[firs, last - 1] |
|first = mid + 1<br>last = mid    |<font color="red">[first, mid) mid [mid + 1, last)</font>|[first, mid] mid [mid + 1, last - 1]，mid 重合|
|first = mid<br>last = mid - 1    |[first, mid - 1) mid [mid, last)，mid - 1 遗漏|[first, mid - 1] mid [mid, last - 1]，mid 重合|
|first = mid <br> last = mid      |[first, mid) mid [mid, last)，mid 重合|[first, mid] mid [mid, last - 1]，mid 重合|
|first = mid + 1<br>last = mid - 1|[first, mid - 1) mid [mid + 1, last)，mid - 1 遗漏|<font color="red">[first, mid - 1] mid [mid + 1, last - 1]</font>|

&#8195;&#8195;所以在非递减序列 test 中查找 val 时下面两种都是正确的，即使区间为空、答案不存在、有重复元素、搜索开/闭的上/下界也同样适用：
```python
def search_val(array, first, last, value):
    while first < last:
        mid = first + (last - first)//2  # 防溢出
        if array[mid] < value:
            first = mid + 1
        else:
            last = mid
    return first # last也行，因为[first, last)为空的时候它们重合
    
def search_val(array, first, last, value):
    while first <= last:
        mid = first + (last - first)//2  # 防溢出
        if array[mid] < value:
            first = mid + 1
        else:
            last = mid - 1
    return first # last也行，因为[first, last)为空的时候它们重合
```

### 注意
&#8195;&#8195;当我们使用左开右闭区间 `(first, last]` 时，闭区间在右侧！算中点时应从闭区间一侧向中心靠拢：
```
mid = last - (last - first) / 2
```
&#8195;&#8195;以确保区间长度为1时，`mid = last` 仍在 `(last - 1, last]` 区间内，如果依然用如下 mid，那么此时 `mid = first` 就超出 `(first, last]` 范围了，要么溢出要么死循环！ 
```
mid = first + (last - first) / 2
``` 

## 参考
[STL中的lower_bound（）和upper_bound（）](https://blog.csdn.net/qq_36172505/article/details/77380559)
[二分查找有几种写法？它们的区别是什么？ - Jason Li的回答](https://www.zhihu.com/question/36132386/answer/530313852)

