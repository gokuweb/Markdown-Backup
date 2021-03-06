---
title: Python 取整
categories:
  - 算法
tags:
  - int
  - roud
abbrlink: 3a91dcd1
date: 2019-06-03 22:51:34
---
&#8195;&#8195;取整在我们的理解中一般就是四舍五入，代码实现过程中也有很多取整函数，但是这取整些函数大部分并非我们常规理解的四舍五入法，而是银行进位法，比如 `round(2.5)` 的结果其实是 2，这里总结一下 Python 中的几种取整方法。
<!-- more -->

&#8195;&#8195;先看一个奇怪的现象：
```python
if __name__ == '__main__':
    print("round(1.5):", round(1.5))
    print("round(2.5):", round(2.5))
    
# round(1.5): 2
# round(2.5): 2
```
&#8195;&#8195;四舍五入 2.5 不应该是 3 吗？这就要提到”银行进位法“，其实在 VB、JS、python 等语言中 `Round()` 函数都是采用”银行进位法“，而非我们以前学的”四舍五入法“。


### 四舍五入法
&#8195;&#8195;没什么好说的，就是传统的四舍五入，保留位的后一位小于 4 则舍，大于等于 5 则进：
> 1. 例如 1.5 保留整数位是 2 ； 2.4 保留整数位是 2 。
> 2. 例如 -1.5 保留整数位是 -2 ； -2.4 保留整数位是 -2 。

&#8195;&#8195;对于负数其实就是先对其取绝对值四舍五入，再加上负号。也可以理解为在数轴上取离得近的那个点。

### 银行进位法
&#8195;&#8195;银行进位法（Banker's Rounding），也就是 **四舍六入五取偶** ：
> 1. 保留位的后一位小于 4 则舍。例如 5.214 保留两位小数为 5.21。
> 2. 保留位的后一位大于 5 则进。例如 5.216 保留两位小数为 5.22。
> 4. 保留位的后一位等于 5 ，且其后 **无** 数字，则要再看 5 的 **前一位**，该数字是 **奇数** 则进，如果是 **偶数** 则舍，例如 5.215 保留两位小数为 5.22 ； 5.225 保留两位小数为 5.22 。
> 3. 保留位的后一位等于 5 ，且其后 **有** 数字，则无论其前一位是奇还是偶都要进，例如 5.2254 保留两位小数为 5.23 。

### python 中取整

**int()**
&#8195;&#8195;向 0 取整。
```   
>>> int(-2.5), int(-1.5), int(1.5), int(2.5) 
-2 -1 1 2
```

**round()**
&#8195;&#8195;银行进位法，四舍六入五取偶。
```
>>> round(-2.5), round(-1.5), round(1.5), round(2.5)
-2 -2 2 2
```

**math.floor()**
&#8195;&#8195;向下取整，找大的。
```
import math
>>> math.floor(-2.5), math.floor(-1.5), math.floor(1.5), math.floor(2.5)
-3 -2 1 2
```

**math.ceil()**
&#8195;&#8195;向上取整，找小的。
```
import math
>>> math.ceil(-2.5), math.ceil(-1.5), math.ceil(1.5), math.ceil(2.5)
-2 -1 2 3
```

** 斜杠 / **
&#8195;&#8195;除法 `/` 总是返回一个浮点数。
```
>>> -2.5/2， -1.5/2， 1.5/2， 2.5/2
    -1.25     -0.75   0.75    1.25
```

** 双斜杠// **
&#8195;&#8195;地板除 `//`，只保留整数部分，向下取整。
```
>>> -2.5//2， -1.5//2， 1.5//2， 2.5//2
     -2.0       -1.0      0.0      1.0
```
&#8195;&#8195;`//` 得到的并不一定都是整数类型的数，它与分母分子的数据类型有关系，如果分子或分母有浮点型，结果就是浮点型：
```
>>> 7//2
3
>>> 7.0//2
3.0
>>> 7//2.0
3.0
```

&#8195;&#8195; `x // y` 等价于 `round((x - fmod(x, y)) / y)`，`//` 其实是就是（被除数 - 余数）/除数。有个点可以注意下：
```
>>> 1 / 0.05
20.0
>>> 1 // 0.05
19.0
```

### 参考
[奇进偶舍](https://baike.baidu.com/item/%E5%A5%87%E8%BF%9B%E5%81%B6%E8%88%8D/2237210?fr=aladdin)
[为什么Python中//和math.floor运算结果会不同](https://zhidao.baidu.com/question/266689520030111845.html)
