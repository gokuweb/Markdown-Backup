---
title: 递归、动态规划和贪心算法
categories:
  - 算法
tags:
  - dp
  - greedy
  - recursion
abbrlink: c0fd6438
date: 2019-05-24 09:42:25
---

&#8195;&#8195;未完待续

<!-- more -->

## 概念
&#8195;&#8195;不管是递归、动态规划还是贪婪算法都是以分治策略为基础，把一个问题分解为一个个小问题，再合并这些小问题的解从而得到结果。

### 分治策略
&#8195;&#8195;分治策略（Divide and Conquer）将原问题分解为若干个规模较小但类似于原问题的子问题（Divide），递归的求解这些子问题（Conquer），然后再合并这些子问题的解来建立原问题的解。因为在求解大问题时，需要递归的求小问题，因此一般用递归的方法实现，即自顶向下。

### 动态规划
&#8195;&#8195;动态规划（Dynamic Programming）的三个核心元素是：
> - 最优子结构；
> - 边界；
> - 状态转移方程式。

&#8195;&#8195;动态规划和分治策略是类似的，也是将一个原问题分解为若干个规模较小的子问题，递归的求解这些子问题，然后合并子问题的解得到原问题的解。和分治的区别在于这些子问题会有重叠，一个子问题在求解后，可能会再次求解，于是我们想到将这些 **子问题的解存储起来** ，当下次再次求解这个子问题时，直接拿过来就是。
&#8195;&#8195;动态规划所解决的问题是分治策略所解决问题的一个子集，只是这个子集更适合用动态规划来解决从而得到更小的运行时间。即 **用动态规划能解决的问题分治策略肯定能解决，只是运行时间长了。因此分治策略一般用来解决子问题相互对立的问题，称为标准分治，而动态规划用来解决子问题重叠的问题。**

动态规划一般由两种方法来实现：
> 1. 自顶向下的备忘录方式，用递归实现；
> 2. 自底向上的方式，用迭代实现。

在「 爬台阶问题 」中只有一个变化维度，比较简单:
> 1. f(10) = f(9) + f(8) 是【最优子结构】  
> 2. f(1) 与 f(2) 是【边界】   
> 3. f(n) = f(n-1) + f(n-2) 【状态转移公式】

### 贪心算法
&#8195;&#8195;贪心算法（Greedy Algorithm）在 **每一步都做出最优的选择** ，希望这样的选择能导致全局最优解。对，只是寄希望，因此 **贪心算法并不保证最终得到最优解** ，但是它对很多问题确实可以得到最优解，而且运行时间更短。由此可见，贪心算法是带有启发性质的算法。那什么时候可以用贪心算法呢？当该问题具有贪心选择性质的时候，我们就可以用贪心算法来解决该问题。 
&#8195;&#8195;贪心选择的性质： **局部最优就是全局最优** 。只要我们能够证明该问题具有贪心选择性质，就可以用贪心算法对其求解。比如对于 0-1 背包问题，我们用贪心算法可能得不到最优解（当然，也可能会得到最优解），但对于部分背包问题，则可以得到最优解，贪心算法可以作为 0-1 背包问题的一个近似算法。

## 递归和迭代
以斐波拉契为例，**递归** 就是在过程或函数里面调用自身：
```python
def fib_recursion(n):
    if n == 1 or n == 2:
        return 1
    else:
        return fibonacci(n-1) + fibonacci(n-2)
```

可以使用 **记忆化搜索** 降低时间复杂度，计算过的就保存起来，下一次直接使用：
```python
s = [-1] * n
def fibonacci2(n):
    global s
    if s[n] != -1:
        return s[n]
    if n == 1 or n == 2:
        return 1
    else:
        s[n] = fibonacci2(n - 1) + fibonacci2(n - 2)
        return s[n]
```

**迭代** 是利用变量的原值推算出变量的一个新值：
```python
def fib_iteration(n):
    a, b = 1, 1
    if n == 1 or n == 2:
        return 1
    for _ in range(n-2):
        a, b = b, a+b
    return b
```
&#8195;&#8195;在 python 中 `a, b = b, a+b` 执行时候 `a` 和 `b` 的赋值是相互独立的， `a=b` 不会影响到 `b=a+b`， `a` 和 `b` 的赋值只受之前的 `a, b = 1, 1` 影响，所以第一次执行完后 `a=1, b=2`，这将影响到下一次循环中 `a` 和 `b` 的赋值。

**如果递归是自己调用自己的话，迭代就是 A 不停的调用 B。**

## 金矿问题
&#8195;&#8195;有一个国家发现了 5 座金矿，每座金矿的黄金储量不同，需要参与挖掘的工人数也不同，分别为200金/3人、300金/4人、350金/3人、400金/5人、500金/5人。参与挖矿工人的总数是 10 人。每座金矿要么全挖，要么不挖，不能派出一半人挖取一半金矿。要求用程序求解出，要想得到尽可能多的黄金，应该选择挖取哪几座金矿？

动态规划中包含三个重要的概念：
> 1. 最优子结构
> 2. 边界
> 3. 状态转移公式

**【最优子结构】有两种情况：**
> 1. 不挖第 5 个金矿，10个工人挖前 4 个金矿所得到的黄金数量。
> 2. 挖第 5 个金矿，此时已有500金，总数是剩下的 5 人从前 4 个金矿取得的黄金数量 + 500金。

4 金矿的最优选择与 5 金矿的最优选择之间的关系是：
```
MAX[（4 金矿 10 工人的挖金数量），
    （4 金矿 5 工人的挖金数量 + 第 5 座金矿的挖金数量）
]
```

**【边界】 有两个：**
> 1. 当只有 1 座金矿时，只能挖这座唯一的金矿，得到的黄金数量为该金矿的数量。
> 2. 当给定的工人数量不够挖 1 座金矿时，获取的黄金数量为 0 。

**【状态转移公式】：**
&#8195;&#8195;我们把金矿数量设为 n，工人数设为 w，金矿的黄金量设为数组 G[200, 300, 350, 400, 500]，金矿的用工量设为数组 P[3, 4, 3, 5, 5]，数组下标从 0 开始，得到【状态转移公式】：
> 1. 边界值：F(n, w) = 0 ，   (n <= 1, w < p[0])
> 2. F(n,w) = g[0]   (n==1, w >= p[0])
> 3. F(n,w) = F(n-1,w)    (n > 1, w < p[n-1])
> 4. F(n,w) = max(F(n-1,w),  F(n-1,w-p[n-1]) + g[n-1])    (n > 1, w >= p[n-1])

&#8195;&#8195;比如 w 个人挖前 2 个金矿 F(2, w)：
> 1. 当 w < 3 时候不足以挖任何一个金矿，所以金矿产出为 0 ；
> 2. 当 w = 3 时候可以挖第一个金矿，产量为 200 金；
> 3. 当 w = 4 时候要比较挖第二个金矿和不挖第二个金矿哪个产量高，F(2, 4) = max(F(1, 4), F(1, 4-4)+300) = max(200, 0+300) = 300 金，所以此时挖第 2 个金矿产量更多，为 300 金；
> 4. 当 w = 7 时候足够两个一起挖，F(2, 7) = max(F(1, 7), F(1, 7-4)+300) = max(200, 200+300) = 500 金，所以此时产量为 500 金。

### 动态规划-迭代
&#8195;&#8195;可以看出每一行都可由前一行推导出来的。比如 F(4, 8) = max(F(3, 8), F(3, 3)+300)。

|     |1 人|2 人|3 人|4 人|5 人|6 人|7 人|8 人|9 人|10 人|
|-----|----|----|----|----|----|----|----|----|----|----|
|1金矿|  0 |  0 |200 |200 |200 |200 |200 |200 |200 |200 |
|2金矿|  0 |  0 |200 |300 |300 |300 |500 |500 |500 |500 |
|3金矿|  0 |  0 |<font color="red">350</font> |350 |350 |550 |650 |<font color="red">650</font> |650 |850 |
|4金矿|  0 |  0 |350 |350 |400 |550 |650 |<font color="red">750</font> |750 |850 |
|5金矿|  0 |  0 |350 |350 |400 |550 |650 |850 |850 |900 |

&#8195;&#8195;所以我们只需要维护一个列表即可，代码实现：
```python
def get_max_gold(n, w, g, p):
    s = [-1]*(w+1)
    r = s[:]
    for k in range(w + 1):
        s[k] = 0 if k < p[0] else g[0]
    print(s)

    for i in range(n):
        if i == 0:
            continue
        for j in range(w+1):
            if j < p[i]:
                r[j] = s[j]
            else:
                r[j] = max(s[j], s[j-p[i]]+g[i])
        s = r.copy() # 注意浅拷贝和赋值的区别，s = r 是不行的
        print(r)
    return r[w]

if __name__ == '__main__':
    G = [200, 300, 350, 400, 500]
    P = [3, 4, 3, 5, 5]
    G1 = [400, 500, 200, 300, 350]
    P1 = [5, 5, 3, 4, 3]
    print(get_max_gold(4, 10, G, P))
    
# [0, 0, 0, 200, 200, 200, 200, 200, 200, 200, 200]
# [0, 0, 0, 200, 300, 300, 300, 500, 500, 500, 500]
# [0, 0, 0, 350, 350, 350, 550, 650, 650, 650, 850]
# [0, 0, 0, 350, 350, 400, 550, 650, 750, 750, 850]
# 850
```
&#8195;&#8195;时间复杂度是 O(n * w)，空间复杂度是 (w)，当金矿数量不变但是人数很大时，动态规划是没有递归节省时间的。

### 动态规划-备忘录法
&#8195;&#8195;备忘录法也称记忆化搜索，把算过的都保存到字典中，人数和金矿为 key，挖到的金矿总数为 value，格式 {(N, W), F(N, W)}，每次调用之前先检查字典中有没有对应的值，有就直接使用无需再计算：
```python
def get_max_gold2(n, w, g, p):
    global num, memoization
    if memoization.get((n, w), None):
        return memoization[n, w]
    num += 1
    if n <= 1 and w < p[n-1]:
        memoization[n, w] = 0
        return 0
    if n == 1 and w >= p[n-1]:
        memoization[n, w] = g[n-1]
        return g[n-1]
    if n > 1 and w < p[n-1]:
        memoization[n, w] = get_max_gold2(n-1, w, g, p)
        return memoization[n, w]
    else:
        memoization[n, w] = max(get_max_gold2(n-1, w, g, p), get_max_gold2(n-1, w-p[n-1], g, p)+g[n-1])
        return memoization[n, w]

if __name__ == '__main__':
    N = 5
    W = 10
    num = 0
    memoization = {}
    G = [200, 300, 350, 400, 500]
    P = [3, 4, 3, 5, 5]
    G1 = [400, 500, 200, 300, 350]
    P1 = [5, 5, 3, 4, 3]
    print(get_max_gold2(N, W, G, P))
    print(num)
    print(memoization)
```

重复的部分将不再计算：
![gold-mine](/imgs/gold-mine.png)

### 普通递归
&#8195;&#8195;递归就是把一个大问题分解成若干小问题，步骤如下：

> 1. 找到如何将大问题分解为小问题的规律
> 2. 通过规律写出递推公式
> 3. 通过递归公式的临界点推敲出终止条件

代码：
```python
def get_max_gold(n, w, g, p):
    global num
    num += 1
    if n <= 1 and w < p[n-1]:
        return 0
    if n == 1 and w >= p[n-1]:
        return g[n-1]
    if n > 1 and w < p[n-1]:
        return get_max_gold(n-1, w, g, p)
    else:
        return max(get_max_gold(n-1, w, g, p), get_max_gold(n-1, w-p[n-1], g, p)+g[n-1])

if __name__ == '__main__':
    num = 0
    G = [200, 300, 350, 400, 500]
    P = [3, 4, 3, 5, 5]
    G1 = [400, 500, 200, 300, 350]
    P1 = [5, 5, 3, 4, 3]
    print(get_max_gold(4, 10, G, P))
    print(num)
    
# 850
```

## 参考

[递归、分治策略、动态规划以及贪心算法之间的关系](https://blog.csdn.net/tyhj_sf/article/details/53969072)
[漫画：什么是动态规划？（整合版）](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653190796&idx=1&sn=2bf42e5783f3efd03bfb0ecd3cbbc380&chksm=8c990856bbee8140055c3429f59c8f46dc05be20b859f00fe8168efe1e6a954fdc5cfc7246b0&scene=21#wechat_redirect)
[看动画轻松理解「递归」与「动态规划」](https://mp.weixin.qq.com/s/kFaJ_aYV7o-_8Ql3w4o1GA)


## 时间复杂度

常见的时间复杂度量级

常数阶O(1)

线性阶O(n)

平方阶O(n²)

对数阶O(logn)

线性对数阶O(nlogn)

[冰与火之歌：「时间」与「空间」复杂度](https://www.cnblogs.com/fivestudy/p/10131495.html)


## 快排
递归树中节点数就是代码计算的调用次数，调用次数是 2^0 + 2^1 + 2^2 + …… + 2^n ，也就是 2^(n+1) - 1 。

类似归并排序的递归树深度是 log 以 n 为底 2 的对数再加 1，快排也是这样，比如 n=8 只会产生 8，4，2，1这四层的节点，相应的第一层要比较 n 次，第二层要根据原始数据的排序情况决定，综合平均情况下快排的时间复杂度是 nlogn 。

[快速排序（过程图解）](https://blog.csdn.net/adusts/article/details/80882649)

