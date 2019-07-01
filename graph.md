---
title: 图论（Graph）
categories:
  - 算法
tags:
  - BFS
  - DFS
  - MST
  - Union-Find
abbrlink: 2fc1a733
date: 2019-06-29 21:31:34
---
&#8195;&#8195;图是一种比线性表和树更为复杂的数据结构，本篇文章较长，涉及图的遍历、最小生成树、最短路径、拓扑排序、关键路径、并查集等，每章节都附有具体的 Python 代码实现。
<!-- more -->

## 图

### 基本概念

&#8195;&#8195;在线性表中，数据元素之间仅有线性关系，每个数据元素（除第一个和最后一个外）只有一个直接前驱和一个直接后继；

&#8195;&#8195;在树形结构中，数据元素之间有着明显的层次关系，并且每个数据元素只与上一层中的一个元素（其双亲结点）及下一层的多个元素（孩子节点）相关；

&#8195;&#8195;而在图形结构中，节点之间的关系是任意的，图中任意两个数据元素之间都有可能相关。



图 G 由两个集合 V 和 E 组成，定义为 G = (V，E)，图可以无边，但至少包含一个顶点：

> 1. 一组顶点：通常用 V (vertex) 表示顶点集合。
> 2. 一组边：通常用 E (edge) 表示边的集合。



图可以分为有向图和无向图，在图中：

> 1. $(v, w)$ 表示 **无向边**，即 $v$ 和 $w$ 是互通的。对于无向图，顶点的度表示以该顶点作为一个端点的边的数目。
> 2. $<v, w>$ 表示 **有向弧**，无向图中叫边，有向图中叫弧。该弧始于 $v$（称为弧尾 Tail 或初始点 Initial node），终于 $w$（称为弧头 Head 或终端点 Terminal node）。对于有向图，顶点的度分为入度和出度。



图中顶点度的概念：

> 1. 度 (Degree) ：所有与顶点 V 连接的边数之和。
> 2. 入度 (Indegree) ：存在于 **有向图** 中，以某顶点为 **弧头**，**终止于** 该顶点的弧的数目称为该顶点的入度。
> 3. 出度 (Outdegree) ：存在于 **有向图** 中，以某顶点为 **弧尾**，**起始于** 该顶点的弧的数目称为该顶点的出度。



图可以分为有权图和无权图：

> 1. 有权图：每条边具有一定的权重 (weight) ，通常是一个数字，**带权值的连通图称为网**。
>
> 2. 无权图：每条边均没有权重，也可以理解为权为 1 。
>
> 3. 不管是无向图还是有向图，顶点数 n，边数 e 和顶点的度数关系是：
>
> $$
> e = \frac{1}{2} \sum^{n}_{i=1} D(v_i)
> $$
>



图可以分为连通图和非连通图：

> 1. 连通图：所有的点都有 **路径** 相连，这需要至少有 n-1 条边。**路径是从一个顶点到另一顶点途径的所有顶点组成的序列**，若路径中 **各顶点都不重复则此路径又被称为“简单路径”**。
> 2. 非连通图：存在某两个点没有路径相连。
> 3. 有向图中的连通图称为强连通图。



完全图，每个顶点都与除自身之外的其他顶点 **恰有一条边** 相连就是完全图：

> 1. 若 G 是无向图，则 $0 ≤ e ≤ n(n-1) / 2$，恰有 $n(n-1)/2$ 条边的无向图称无向完全图(Undireet-ed Complete Graph)，注意边和路径的区别。
> 2. 若 G 是有向图，则 $0 ≤ e ≤ n(n-1 ,恰有 $n(n-1)$ 条弧的有向图称为有向完全图(Directed Complete Graph)。



稀疏图和稠密图：

> 1. 有很少的边或弧（e < nlogn）的图就称为稀疏图（Sparse graph），稀疏图适合用邻接表存储。
>
> 2. 反之，则称此图为稠密图（Dense graph），稠密图适合用邻接矩阵存储。



极大连通子图和极小连通子图，**极大极小是针对边数**：

> 1. 极大连通子图，包含最多边的连通子图。对于连通图，极大连通子图其实就是它本身。
> 2. 极小连通子图，包含最少边的连通子图。对于连通图，极小连通子图其实就是它的生成树，多加一条边就会产生环，减少一条边就无法构成完整的生成树。



&#8195;&#8195;**连通分量**（Connected Component）指的就是无向图中的极大连通子图（注意不是最大，最大子图只有一个）。任何连通图的连通分量只有其自身一个，非连通的无向图有多个连通分量，如下 $G_3$ 是非连通图，有三个连通分量。有向图中的极大强连通子图称为有向图的强连通分量。

![connected-component](\imgs\connected-component.png)



&#8195;&#8195;**生成树**，一个连通图的生成树是一个极小连通子图，它含有图中全部顶点，但只有足以构成一棵树的 n - 1 条边，比如上如图 (b) 中第一个图把 J 到 L 之间的边取消，就是 $G_3$ 的一个生成树。



[数据结构(C语言版)].严蔚敏]

[《数据结构与算法》——极大连通子图、极小连通子图（一次性解决）](https://blog.csdn.net/lenmonhacker/article/details/84070887)



### 图的存储结构

&#8195;&#8195;图的常用存储结构有邻接表、邻接矩阵、邻接多重表、十字链表，我们主要看看邻接表和邻接矩阵：

> 1. 邻接矩阵，边无权重的话，如果两顶点之间有边就用 1 表示，否则用 0；有权重的话，如果两顶点之间有边就用 权值 表示，否则用无穷大 $\infty$ 表示。无向图有对称性，可以采用压缩寸尺的方式只存入矩阵下三角（或上三角）元素。
> 2. 邻接表，对图中每个顶点建立一个单链表，代表了这个顶点和与之相邻的顶点的关系，如下图邻接表，第一条单链表发意思是与 $v_1$ 相邻的顶点分别是下标为 2 和 1 的顶点，也就是 $v_3$ 和 $v_2$ 。



![graph-struct](\imgs\graph-struct.png)



&#8195;&#8195;邻接矩阵一般用二维数组实现，以二维数组表示有 $n$ 个顶点的图时，需存放 $n$ 个顶点的信息和 $n^2$ 个弧信息的存储量。构造一个具有 $n$ 个顶点 $e$ 条边的无向网 G 时间复杂度是 $O(n^2 + e \cdot n)$ ，其中对邻接矩阵 G 的初始化耗费了 $O(n^2)$ 的时间。

&#8195;&#8195;在邻接表上容易找到任一顶点的第一个邻接点和下一个邻接点，但要判断任意两个顶点之间是否有边活着弧，需要分别搜索这两个链表，此时不及邻接矩阵方便。



## 图的遍历

&#8195;&#8195;从图的某一顶点出发访遍图中其余顶点，且使每一个顶点仅被访问一次，这个过程就叫图的遍历（Traversing Graph）。图的遍历算法是求解图的连通性问题、拓扑排序和求关键路径等算法的基础。

### DFS 算法

&#8195;&#8195;深度优先搜索算法（Depth-First-Search，DFS），类似于树的前序遍历：

> 1. 初始状态是图中所有顶点未被访问过，首先访问出发点 v，并将其标记为已访问过。
> 2. 然后依次从 v 的未被访问过的邻接点出发做深度优先遍历， 直到图中所有和 v 有路径相通的顶点都被访问到。
> 3. 若此时图中仍有顶点未被访问（这些顶点和 v 不连通），则另选一个未曾访问的顶点作为起点，重复上述步骤，直到图中所有顶点都被访问到为止。

&#8195;&#8195;如果采用邻接矩阵存储，则时间复杂度为 $O(n^2)$；当采用邻接表时时间复杂度为 $O(n + e)$。

&#8195;&#8195;DFS 的实现有递归和迭代两种方法，我们看一下迭代的步骤。先定义一个列表 stack 和集合 visited，列表用于模拟栈，每个未访问过的节点都要入栈，后进先出；visited 是一个集合，用于存储访问过的节点，集合本来是无序的，所以集合中的元素顺序未必是节点的访问顺序，但栈一定是按照元素访问顺序弹出的。DFS 过程中会产生很多种路径，我们这里模拟一下 A B E F C D 的路径，初始把 A 入栈，visited 空：

> 1. 此时栈顶元素是 A，弹出 A ，如果 A 不在 visited 中（即未被访问过），则把 A 加入 visited，查找与 A 有边的顶点得到 C B，若 C B 不在 visited 中，就把 C B 入栈；
> 2. 此时栈顶元素是 B，弹出 B，如果 B 不在 visited 中，则把 B 加入 visited，查找与 B 有边的顶点得到 A D E，D E 不在 visited 中，就把 D E 入栈；
> 3. 此时栈顶元素是 E，弹出 E ，如果 E 不在 visited 中，则把 E 加入 visited，查找与 E 有边的顶点得到 B F，F 不在 visited 中，就把 F 入栈；
> 4. 此时栈顶元素是 F，弹出 F ，如果 F 不在 visited 中，则把 F 加入 visited，查找与 F 有边的顶点得到 C E，C 不在 visited 中，就把 C 入栈（所以一个元素可能会多次入栈出栈，我么们只看是否被访问过）；
> 5. 此时栈顶元素是 C，弹出 C ，如果 C 不在 visited 中，则把 C 加入 visited，查找与 C 有边的顶点得到 A F，A F 都在 visited 中，栈未空，那就弹出当前栈顶元素；
> 6. 此时栈顶元素是 D，弹出 D ，如果 D 不在 visited 中，则把 D 加入 visited，查找与 D 有边的顶点得到 B E，B E 都在 visited 中，栈未空，那就弹出当前栈顶元素；
> 7. 此时栈顶元素是 C，弹出 C ，发现 C 在 visited 中，不做处理，栈空，结束循环。

![DFS1](\imgs\DFS1.png)



迭代法一：

```python
def dfs_iterate1(graph, start):
    stack = [start]
    visited = set()
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            for w in graph[vertex]:
                if w not in visited:
                    stack.append(w)
            #print(vertex) # 按顺序输出访问节点
    return visited # 集合无序，只代表这些节点访问过

if __name__ == '__main__':
    raph = {'A': ['B', 'C'],
            'B': ['A', 'E', 'D'],
            'C': ['A', 'F'],
            'D': ['B'],
            'E': ['B', 'F'],
            'F': ['C', 'E']}
    print(dfs_recursion(graph, "A"))
```



迭代法二：

```python
def dfs_iterate2(graph, start):
    visited, stack = set(), [start]
    while stack:
        vertex = stack.pop()
        if vertex not in visited:
            visited.add(vertex)
            stack.extend(graph[vertex] - visited)
            yield vertex

if __name__ == '__main__':
    graph = {'A': set(['B', 'C']),
             'B': set(['A', 'E', 'D']),
             'C': set(['A', 'F']),
             'D': set(['B']),
             'E': set(['B', 'F']),
             'F': set(['C', 'E'])}
    print(list(dfs_iterate2(graph, "A")))
```



递归实现：

```python
def dfs_recursion(graph, start, visited = None):
    if visited is None:
        visited = set()
    visited.add(start)
    #print(start)   # 按顺序输出访问节点
    for v in graph[start]:
        if v not in visited:
            dfs_recursion(graph, v, visited)
    return visited  # 输出所有访问过节点，但集合无序
```





### BFS 算法

&#8195;&#8195;广度优先搜索算法（Breadth-First-Search，缩写为 BFS），类似于树的层次遍历：

> 1. 初始状态是图中所有顶点未被访问过，首先访问出发点 v，并将其标记为已访问过。
> 2. 然后依次访问 v 的各个未曾访问过的邻接点，然后分别从这些邻接点出发 **依次** 访问它们的邻接点， 直到图中所有和 v 有路径相通的顶点都被访问到。
> 3. 若此时图中仍有顶点未被访问（这些顶点和 v 不连通），则另选一个未曾访问的顶点作为起点，重复上述步骤，直到图中所有顶点都被访问到为止。

&#8195;&#8195;如果采用邻接矩阵存储，则时间复杂度为 $O(n^2)$；当采用邻接表时时间复杂度为 $O(n + e)$。

迭代法一：

```python
def bfs_iterate1(graph, start):
    visited, q = set(), [start]
    while q:
        vertex = q.pop(0)
        if vertex not in visited:
            visited.add(vertex)
            for w in graph[vertex]:
                if w not in visited:
                    q.append(w)
            #print(vertex)
    return visited
```



迭代法二：

```python
def bfs_iterate2(graph, start):
    visited, q = set(), [start]
    visited.add(start)
    while q:
        vertex = q.pop(0)
        for w in graph[vertex]:
            if w not in visited:
                visited.add(w)
                q.append(w)
        #print(vertex)
    return visited
```



[[Python] BFS和DFS算法（第1讲）](https://www.bilibili.com/video/av25761720/)



## 最小生成树

&#8195;&#8195;最小生成树(Minimum Spanning Trees MST)，包含图中所有顶点且没有环的子图就是生成树，一个连通图可能有多个生成树，权值和最少的就是最小生成树。对于非连通无向图来说，它的每一连通分量同样有最小生成树，它们的并被称为最小生成森林。最小生成树的算法有 Prim 和 Kruskal，它们的核心思想都是贪心算法。



### Prim 算法

&#8195;&#8195;Prim（普里姆算法）算法每次选择一个与当前 **顶点集** 最近的一个顶点，并将两顶点之间的边加入到树中，直到所有顶点都被访问。该树最开始只有一个顶点，然后会添加  V-1 个边，时间复杂度为  $O(n^2)$。



**算法步骤：**

> 1. 假如从 $v_0$ 出发，初始化一个列表 dist 保存 $v_0$ 到其他各边的最短距离。
> 2. 从 dist 中找出未被访问过的最短的一条边，获取这个顶点 minnode并将其标记为已访问过。
> 3. 更新 dsit 为从 minnode 到其余未访问过顶点的最短距离。
> 4. 重复 2，3 步骤直到所有顶点被访问过。



方法一：

```python
def prim1(graph):
    global inf
    vertex_num = len(graph[0])
    dist = [inf] * vertex_num
    visited = [0] * vertex_num
    dist[0], mstpath = 0, []
    totalweight = 0
    for _ in range(vertex_num):
        minnode, minweight = None, inf
        for i in range(vertex_num):
            if visited[i] == 0 and dist[i] < minweight:
                minnode, minweight = i, dist[i]

        totalweight += minweight
        visited[minnode] = 1
        for i in range(vertex_num):
            if visited[i] == 0 and dist[i] > graph[minnode][i]:
                dist[i] = graph[minnode][i]
        mstpath.append(minnode)
    #print("weight:", dist)

    return mstpath, dist

if __name__ == '__main__':
    inf = float('inf')
    # 对角线，即自己到自己的权值用 0 或 inf 不影响结果，这里就不改了
    graph1_2 = [[inf, 10, inf, inf, inf, 11, inf, inf, inf],
                [10, inf, 18, inf, inf, inf, 16, inf, 12],
                [inf, 18, inf, 22, inf, inf, inf, inf, 8],
                [inf, inf, 22, inf, 20, inf, inf, 16, 21],
                [inf, inf, inf, 20, inf, 26, 7, 19, inf],
                [11, inf, inf, inf, 26, inf, 17, inf, inf],
                [inf, 16, inf, inf, 7, 17, inf, 19, inf],
                [inf, inf, inf, 16, 19, inf, 19, inf, inf],
                [inf, 12, 8, 21, inf, inf, inf, inf, inf]]

    mst, weight = prim1(graph1_2)
    print("mst:%s \nweight:%s" % (mst, weight))
    
# mst:[0, 1, 5, 8, 2, 6, 4, 7, 3] 
# weight:[0, 10, 8, 16, 7, 11, 16, 19, 12]
```

&#8195;&#8195;初始化 `dist = [0, inf, inf, inf...]` ，意思是 $v_0$ 到自己的最短距离是 0，到其它顶点的最短距离都是无穷大。 `visited = [0, 0, 0, 0...]` ，意思是所有顶点未被访问过。对于 n 个顶点的图，大循环是 n - 1 次，因为从 $v_0$ 出发一次选一个顶点，n - 1 次就能把所有顶点都访问到，里面是两个循环，第一个循环找距离当前顶点最近的顶点，第二个循环把 dist 更新为从这个顶点出发到其他未访问过顶点的最短距离。比如上面第一次大循环中，第一个 for 找到了 $v_0$，第二个 for 把 dist 更新为 $v_0$ 到其他未访问过顶点的最短距离，即 `dist = [0, 10, inf, inf, inf, 11, inf, inf, inf]` ；第二次大循环中，第一个 for 找到了 $v_1$，第二个 for 把 dist 更新为 $v_1$ 到除 $v_0$ 外其他顶点的最短距离，即 `dist = [0, 10, 18, inf, inf, 11, 16, inf, 12]` ，以此类推；可以看出来找到 $v_1$ 后更新距离，其实就是把 graph[0] 和 graph[1] 取最小的值保存到 dist。



方法二：

```python
def prim2(graph, start):
    global inf
    visited, totalweight = [start], 0
    currentnode, mstpath = None, []
    while len(visited) < len(graph[start]):
        minweight, minnode = inf, None
        for vertex in visited:
            for neighbour in graph[vertex]:
                if neighbour not in visited and \
                        graph[vertex][neighbour] < minweight:
                    currentnode, minnode, minweight = vertex, \
                        neighbour, graph[vertex][neighbour]
        mstpath.append((currentnode, minnode))
        visited.append(minnode)
        totalweight += minweight
    return visited, mstpath
    
if __name__ == '__main__':
    inf = float('inf')
    graph2_1 = {'A': {'A': 0,   'B': 1,   'C': 12,  'D': inf, 'E': inf, 'F': inf},
                'B': {'A': 1,   'B': 0,   'C': 9,   'D': 3,   'E': inf, 'F': inf},
                'C': {'A': 12,  'B': 9,   'C': 0,   'D': 4,   'E': 5,   'F': inf},
                'D': {'A': inf, 'B': 3,   'C': 4,   'D': 0,   'E': 13,   'F': 15},
                'E': {'A': inf, 'B': inf, 'C': 5,   'D': 13,   'E': 0,   'F': 4},
                'F': {'A': inf, 'B': inf, 'C': inf, 'D': 15,   'E': 4,   'F': 0}
                }
    visited, mstpath = prim2(graph2_1, "A")
    print("mst:{0} \nmstpath:{1}".format(visited, mstpath))
    
# mst:['A', 'B', 'D', 'C', 'E', 'F'] 
# mstpath:[('A', 'B'), ('B', 'D'), ('D', 'C'), ('C', 'E'), ('E', 'F')]   
```

&#8195;&#8195;这个时间复杂度比较高，但是感觉逻辑更清楚些。初始 visited 里面有一个起始顶点  $v_0$ ，找到距离  $v_0$ 最近的顶点  $v_1$ 加入 visited；现在 visited 里面有两个顶点，继续找距离  $v_0, v_1$ 最近的未被访问过的顶点，找到后再加入 visited，直到 visited 里面包含了所有顶点。



### Kruskal 算法

&#8195;&#8195;Kruskal（克鲁斯卡尔）算法是先按照边的权重（从小到大）进行排序，然后再依次加入生成树中，若加入该边会形成环则不加入该边。直到树中含有 V-1 条边为止。这些边组成的就是该图的最小生成树。Kruskal 算法的时间复杂度为 O(Elog E)。

**算法步骤：**

> 1. 将图中所有的边按权值从小到大排序；
> 2. 从权值最小的边开始，如果这条边的两个顶点不在同一集合，则选定这条边；
> 3. 重复 2 ，直至图中所有的节点都在同一集合。
> 4. 从 dist 中找出未被访问过的最短的一条边，获取这个顶点 minnode并将其标记为已访问过。
> 5. 更新 dsit 为从 minnode 到其余未访问过顶点的最短距离。
> 6. 重复 2，3 步骤直到所有顶点被访问过。



**方法一：**

&#8195;&#8195;判断两个顶点是否在同一集合，通常使用并查集，（后面章节有详细介绍并查集），用并查集法实现的代码如下：

```python
# 常规版：
class solution():
    def kruska(self, graph, vertexes):
        parent = {}
        for vertex in vertexes:
            parent[vertex] = vertex
        graph.sort(key=self.cmp_weight)

        for edge in graph:
            vertex1, vertex2, weight = edge
            x_root = self.find_father(vertex1, parent)
            y_root = self.find_father(vertex2, parent)
            if x_root != y_root:
                print("edge:", edge)
                self.merge_node(x_root, y_root, parent)
        print(parent)

    def cmp_weight(self, elem):
        return elem[2]

    def find_father(self, x, parent):
        while x != parent[x]:
            x = parent[x]
        return x

    def merge_node(self, x, y, parent):
        parent[y] = x

if __name__ == '__main__':
    vertexes = ["A", "B", "C", "D", "E", "F", "G", "H", "I"]
    graph3 = [["A", "B", 10], ["A", "F", 11], ["B", "C", 18], ["B", "G", 16],
              ["B", "I", 12], ["C", "D", 22], ["C", "I", 8],  ["D", "E", 20],
              ["D", "H", 16], ["D", "I", 21], ["E", "F", 26], ["E", "G", 7],
              ["E", "H", 19], ["F", "G", 17], ["G", "H", 19]] 
    s = solution()
    s.kruska(graph3, vertexes)

{'A': 'A', 'B': 'A', 'C': 'A', 'D': 'A', 'E': 'A', 'F': 'A', 'G': 'E', 'H': 'D', 'I': 'C'}
```

&#8195;&#8195;parent 仅用于记录各顶点的根节点，这个树形结构跟最终的最小生成树没关系，parent 是一个三层的树形结构，可以做一些优化：

```python
# 优化版
class solution():
    def kruska(self, graph, vertexes):
        parent, rank = {}, {}
        for vertex in vertexes:
            parent[vertex] = vertex
            rank[vertex] = 1
        graph.sort(key=self.cmp_weight)

        for edge in graph:
            vertex1, vertex2, weight = edge
            x_root = self.find_father(vertex1, parent)
            y_root = self.find_father(vertex2, parent)
            if x_root != y_root:
                print("edge:", edge)
                self.merge_node(x_root, y_root, parent, rank)
        print(parent)

    def cmp_weight(self, elem):
        return elem[2]

    def find_father(self, x, parent):
        while x != parent[x]:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def merge_node(self, x, y, parent, rank):
        if rank[x] > rank[y]:
            parent[y] = x
        elif rank[x] < rank[y]:
            parent[x] = y
        else:
            parent[y] = x
            rank[x] += 1
            
if __name__ == '__main__':
    vertexes = ["A", "B", "C", "D", "E", "F", "G", "H", "I"]
    graph3 = [["A", "B", 10], ["A", "F", 11], ["B", "C", 18], ["B", "G", 16],
              ["B", "I", 12], ["C", "D", 22], ["C", "I", 8],  ["D", "E", 20],
              ["D", "H", 16], ["D", "I", 21], ["E", "F", 26], ["E", "G", 7],
              ["E", "H", 19], ["F", "G", 17], ["G", "H", 19]] 
    s = solution()
    s.kruska(graph3, vertexes)
    
{'A': 'A', 'B': 'A', 'C': 'A', 'D': 'A', 'E': 'A', 'F': 'A', 'G': 'A', 'H': 'A', 'I': 'A'}
```

&#8195;&#8195;现在 parent 变成了一层的树形结构，遍历时候效率更高。下面是网上找的测试用例：

![mst](\imgs\mst.png)

**方法二：**

&#8195;&#8195;并查集是查找每个顶点的根节点，如果两个顶点有相同的根节点，那就说明这条边加进去会产生环，如果根节点不同，就说明这条边是对的。下面我们用另一种方法实现，依然是不断取出已经排好队的边，定义一个字典，key 对应这条边的顶点，value 对应这个顶点可达的所有其他顶点，这样如果两个顶点在同一集合，那它们的 value 必然相同：

```python
def kruskal(graph, vertexes):
    parent = {}
    for vertex in vertexes:     # 初始化 parent
        parent[vertex] = [vertex]
    graph3.sort(key=lambda item: item[2]) # 对边排序

    for edge in graph:     # 如果该边不会产生环，则打印
        vertex1, vertex2, weight = edge
        if parent[vertex1] != parent[vertex2]:
            print(edge)
            for v in parent[vertex2]:
                if v not in parent[vertex1]:
                    parent[vertex1].append(v)
                parent[v] = parent[vertex1]
            #print(parent)

if __name__ == '__main__':
	vertexes = ["A", "B", "C", "D", "E", "F", "G", "H", "I"]
    graph3 = [["A", "B", 10], ["A", "F", 11], ["B", "C", 18], ["B", "G", 16],
              ["B", "I", 12], ["C", "D", 22], ["C", "I", 8],  ["D", "E", 20],
              ["D", "H", 16], ["D", "I", 21], ["E", "F", 26], ["E", "G", 7],
              ["E", "H", 19], ["F", "G", 17], ["G", "H", 19]]

    kruskal(graph3, vertexes)
```



[Prim's Algorithm for MST(with Code Walkthrough) ](https://youtu.be/eB61LXLZVqs)

[【数据结构算法】图（六）：基于邻接矩阵的最小生成树（prim算法）Python实现](https://blog.csdn.net/qiu931110/article/details/80522692)

[Python图论算法（三）——kruskal](<https://blog.csdn.net/u010352695/article/details/44679889>)



## 最短路径

### Dijkstra 算法

&#8195;&#8195;Dijkstra（迪杰斯特拉）算法是典型的 **单源最短路径算法** （SSSP：Single-Source Shortest Path），用于计算从 **一个顶点到其余各顶点** 的最短路径，主要特点是以起始点为中心向外层层扩展，直到扩展到终点为止，时间复杂度为 $O(n^2)$。Dijkstra 算法 **不适用于有负权值** 的情况，因为 Dijkstra 是 **贪心算法**，贪心算法成功的前提是局部最优解即全局最优解，但存在负权的话不能保证局部最优解就是全局最优解，负权值可以使用 Bellman-Ford 算法。

**算法思想：**

&#8195;&#8195;设 G=(V, E) 是一个带权有向图，把图中顶点集合 V 分成两组，**第一组为已求出最短路径的顶点集合**（用 S 表示），初始时 S 中只有一个源点 $v$，以后每求得一条最短路径 , 就将对应的顶点加入到集合 S 中，直到全部顶点都加入到 S 中就结束了；**第二组为其余未确定最短路径的顶点集合**（用 U 表示），按最短路径长度的递增次序依次把第二组的顶点加入 S 中。在加入的过程中，总保持从源点 $v$ 到 S 中各顶点的最短路径长度不大于从源点 $v$ 到 U 中任何顶点的最短路径长度。此外，每个顶点还对应一个距离，S 中的顶点的距离就是从 $v$ 到此顶点的最短路径长度，U 中的顶点的距离，是从 $v$ 到此顶点的最短路径长度，但是可以以 S 中的顶点为中间顶点。最后这句比较拗口，比如 S 中有 A B，U中有 C D E F，假如 A C 间的权重是 8 但 A B C 这条路径的权重是6，那 U 中 C 对应的长度就是 6 。



**算法步骤：**

> 1. 初始化，用集合 S 存放已求得最短路径的顶点以及这个顶点到源点的距离，用集合 U 存放未求得最短路径的顶点以及这个顶点到源点的距离，这一步主要是把源点 $v$ 放到集合 S 中，源点到自己的距离是 0 ，即 $S＝\{v(0)\}$。U 包含除 $v$ 外的其他顶点，即 $U = \{其余顶点(到源点距离)\}$，若 $v$ 与 U 中顶点 $u$ 有直接的边，则 $<u, v>$ 的权值就是距离，若无直接相连的边，则 $<u, v>$ 的距离用无穷大  $\infty$ 表示。
> 2. 主要做两件事，一是从 U 中选取一个距离源点最小的顶点 $k$ ，把 $k$ 从 U 移到 S 中，此时 $S＝\{v(0), k(v到k距离)\}$。二是更新 U 中各顶点到源点的距离，$k$ 加入 S 之后，就要考虑以 $k$  为中间点的时候，U 中各顶点到源点的距离，如果这个距离更近了，就要替换掉原来的距离，比如下图第二步，本来 B 到 源点 D  的距离是无穷大，但是以 C 为中间点的话这个距离变成了 13 ，所以 B 的距离更新为 13 ，这个更新距离的过程叫松弛。
> 3. 从 U 中选取距离源点最近的顶点，移到 S 中，然后对 U 中每个顶点进行松弛。
> 4. 重复第 3 步，如果集合 U 为空，算法结束。



对比下图理解：

![dijkstra](\imgs\dijkstra.png)



**代码实现：**

```python
def dijkstra(graph, start, goal):
    dist = {}
    predecessor = {}
    path = []
    unseen = graph

    for vertex in graph:
        dist[vertex] = 999
    dist[start] = 0

    # 核心代码
    while unseen:
        minode = None
        for node in unseen:
            if minode is None:
                minode = node
            elif dist[node] < dist[minode]:
                minode = node

        for childnode, weight in graph[minode].items():
            if weight + dist[minode] < dist[childnode]:
                dist[childnode] = weight + dist[minode]
                predecessor[childnode] = minode
        unseen.pop(minode)
        #print(dist, predecessor)

    # 输出 A -> F 最短路径
    currentnode = goal
    while currentnode != start:
        try:
            path.insert(0, currentnode)
            currentnode = predecessor[currentnode]
        except KeyError:
            print("Path not reachable")
            break
            
    print("dist is:", dist)
    print("predecessor is:", predecessor)
    print("path is:", path)

if __name__ == '__main__':
    graph = {'A': {'B': 1, 'C': 12},
             'B': {'C': 9, 'D': 3},
             'C': {'E': 5},
             'D': {'C': 4, 'E': 13, 'F': 15},
             'E': {'F': 4},
             'F': {}
             }

    dijkstra(graph, "A", "F")
```

&#8195;&#8195;python 中可以用 $\pm inf$ 表示正负无穷：`inf = float('inf')`，也可以用一个足够大的数字，如上代码中用的是 999 。其实最核心的代码不到 15 行，这里没有考虑具体边界问题，默认输入都是合法的。



**存在负权时候的最短路径：**

&#8195;&#8195;为什么有负权（注意区别负权和负权环）的情况下 Dijkstra 算法会失效？如下左侧的图是一个经典的反面案例，但实际上在某些情况下这个图可以用 Dijkstra 算法得到正确解：

![dijkstra-negative ](\imgs\dijkstra-negative.png)

&#8195;&#8195;左侧图能否用 Dijkstra 算法得到正确解，取决于代码终止条件，首先图中顶点被分为两组，用 S 表示已求出最短路径的顶点的集合，用 U 表示未确定最短路径的顶点的集合，最终 U 中的顶点都会移动到 S 中，如左图当 U 种只剩下最后一个顶点 B 的时候，考虑下面两种情况：

> 1. 把 B 移动到 S 中，对 B 的各弧进行松弛。
> 2. 把 B 移动到 S 中，如果 B 的邻接点在 U 中，那么对相关的各弧进行松弛。



&#8195;&#8195;很明显第一种情况能正确求出左图的最短路径，第二种情况不能，但无论哪种情况，都无法正确求出右图的最短路径，这时候需要用到 Bellman-Ford 算法。



### Floyd 算法

&#8195;&#8195;Floyd-Warshall（弗洛伊德）算法是典型的 **多源最短路径算法** ，用于计算 **任意两个顶点** 间的最短路径，可以正确处理有向图或负权（但不可存在负权回路）的最短路径问题，同时也被用于计算有向图的传递闭包。其实对每个顶点做 Dijkstra 也可以求得任意两顶点的最短路径，时间复杂度是 $O(n^3)$，但 Floyd-Warshall 算法要简单些，时间复杂度也是 $O(n^3)$，空间复杂度为 $O(n^2)$。



**算法思想：**

&#8195;&#8195;Floyd 算法运用了动态规划的思想。用通俗的语言来描述的话，从顶点 i 到顶点 j 的最短路径不外乎两种可能，直接从 i 到 j ，活着从 i 经过若干个节点 k 到 j。所以，我们假设 `dis(i, j)` 为顶点 v 到顶点 u 的最短路径距离，对于每一个顶点 k，我们检查 `dis(i, k) + dis(k, j) < dis(i, j)` 是否成立，如果成立，证明从 i 到 k 再到 j 的路径比 i 直接到 j 的路径短，我们便设置 `dis(i, j) = dis(i, k) + dis(k, j)`，这样一来，当我们遍历完所有节点 k，`dis(i, j)` 中记录的便是 i 到 j 的最短路径的距离。



**算法步骤：**

> 1. 从任意一条单边路径开始。所有两点之间的距离是边的权，如果两点之间没有边相连，则权为无穷大  $\infty$。
> 2. 对于每一对顶点 $u$ 和 $v$，看看是否存在一个中间顶点 $k$ 使得从 $u$ 到 $k$ 再到$ v$ 比己知的路径更短，如果存在就更新它。



&#8195;&#8195;如现在只允许经过 1 号顶点，求任意两点之间的最短路程，应该如何求呢？只需判断 `e[i][1]+e[1][j]` 是否比` e[i][j]` 要小即可。`e[i][j]` 表示的是从 i 号顶点到 j 号顶点之间的路程。`e[i][1]+e[1][j]` 表示的是从 i 号顶点先到 1 号顶点，再从 1 号顶点到 j 号顶点的路程之和。其中 i 是 0~n-1 循环，j 也是 0~n-1 循环，代码实现如下：
```python
for i in range(n):
    for j in range(n):
        if e[i][j] > e[i][1]+e[1][j]:
            e[i][j] = e[i][1]+e[1][j]
```



&#8195;&#8195;接下来继续求在只允许经过 1 和 2 号两个顶点的情况下任意两点之间的最短路程。如何做呢？我们需要在只允许经过 1 号顶点时任意两点的最短路程的结果下，再判断如果经过 2 号顶点是否可以使得 i 号顶点到 j 号顶点之间的路程变得更短。即判断 `e[i][2]+e[2][j]` 是否比 `e[i][j]` 要小，代码实现为如下：

```python
# 经过1号顶点
for i in range(n):
    for j in range(n):
        if e[i][j] > e[i][1]+e[1][j]:
            e[i][j]=e[i][1]+e[1][j]
# 经过2号顶点
for i in range(n):
    for j in range(n):
        if e[i][j] > e[i][2]+e[2][j]:
            e[i][j]=e[i][2]+e[2][j]
```



**代码实现：**

```python
def floyd(graph):
    for k in graph.keys():
        for i in graph.keys():
            for j in graph[i].keys():
                if graph[i][j] > graph[i][k] + graph[k][j]:
                    graph[i][j] = graph[i][k] + graph[k][j]
    for i in graph.keys():
        print(list(graph[i].values()))

if __name__ == '__main__':
    inf = float('inf')
    graph = {'A': {'A': 0, 'B': 1, 'C': 12, 'D': inf, 'E': inf, 'F': inf},
              'B': {'A': inf, 'B': 0, 'C': 9, 'D': 3, 'E': inf, 'F': inf},
              'C': {'A': inf, 'B': inf, 'C': 0, 'D': inf, 'E': 5, 'F': inf},
              'D': {'A': inf, 'B': inf, 'C': 4, 'D': 0, 'E': 13, 'F': 15},
              'E': {'A': inf, 'B': inf, 'C': inf, 'D': inf, 'E': 0, 'F': 4},
              'F': {'A': inf, 'B': inf, 'C': inf, 'D': inf, 'E': inf, 'F': 0}
              }
    
    floyd(graph)
```



[算法 6：只有五行的 Floyd 最短路算法](http://wiki.jikexueyuan.com/project/easy-learn-algorithm/floyd.html)



### Bellman-Ford 算法

&#8195;&#8195;Bellman - ford（贝尔曼-福特）算法是求含负权图（但无负权环）的单源最短路径的一种算法，效率较低。其原理为连续进行松弛，在每次松弛时把每条边都更新一下，若在 n-1 次松弛后还能更新，则说明图中有负权环，因此无法得出结果，否则就完成，有动态规划的思想。



**算法思想：**
&#8195;&#8195;从任意一条单边路径开始。所有两点之间的距离是边的权，如果两点之间没有边相连，则权为无穷大  $\infty$。 对于每一对顶点 $u$ 和 $v$，看看是否存在一个中间顶点 $k$ 使得从 $u$ 到 $k$ 再到$ v$ 比己知的路径更短，如果存在就更新它。



**算法步骤：**

> 1. 初始化一个字典。键值分别是各顶点和源点到该顶点的权值，将源点到自己的权值设为 0，和源点没有直接相连的设为无穷大（表示不可达）。
> 2. 进行循环，循环次数为 n－1（n 等于图中顶点个数）。在循环内部，以各个顶点为中间点进行松弛（这里有点像 Floyd 算法）。
> 3. 遍历图中所有的边 `edge(u，v)`，如果还存在 `d(v) > d(u) + w(u, v)`，意味着图中有负回路，返回 false 。



**代码实现：**

```python
def bellman_ford(graph, start):
    # Step 1: Prepare the distance and predecessor for each node
    dist, predecessor = dict(), dict()
    for node in graph:
        dist[node], predecessor[node] = float('inf'), None
    dist[start] = 0

    # Step 2: Relax the edges
    for _ in range(len(graph) - 1):
        for node in graph:
            for neighbour in graph[node]:
                # If the distance between the node and the neighbour
                # is lower than the current, store it
                if dist[neighbour] > dist[node] + graph[node][neighbour]:
                    dist[neighbour], predecessor[neighbour] = \
                        dist[node] + graph[node][neighbour], node

    # Step 3: Check for negative weight cycles
    for node in graph:
        for neighbour in graph[node]:
            assert dist[neighbour] <= dist[node] + graph[node][neighbour], \
                "Negative weight cycle."

    return dist, predecessor

if __name__ == '__main__':
    inf = float('inf')
    graph = {'A': {'B': 1, 'C': 0, 'D': 99},
             'B': {'C': 1},
             'C': {},
             'D': {'B': -300},
             }

    distance, predecessor = bellman_ford(graph, start='A')
```



&#8195;&#8195;Bellman-Ford 算法寻找单源最短路径的时间复杂度为 O(V*E) 。为什么要循环 n-1 次？ 因为最短路径肯定是个简单路径，不可能包含回路， 如果包含回路且回路的权值和为正的，那么去掉这个回路就可以得到更短的路径。如果循环 n-1 次后，还有权值不断缩小，说明存在负权环，正权环的总权值每次循环只会不断增加。



[图的最短路径之迪杰斯特拉算法和弗洛伊德算法](https://blog.csdn.net/daaikuaichuan/article/details/80586408)

[Implementation of dijkstra in python](https://youtu.be/IG1QioWSXRI)

[Negative weights using Dijkstra's Algorithm](https://stackoverflow.com/questions/6799172/negative-weights-using-dijkstras-algorithm)

[Python实现图的经典DFS、BFS、Dijkstra、Floyd、Prim、Kruskal算法](https://zhuanlan.zhihu.com/p/61628249)

[python数据结构与算法——图的最短路径（Floyd-Warshall算法](https://www.bbsmax.com/A/KE5Q0ZELJL/)

[[Bellman-Ford algorithm in python](https://gist.github.com/ngenator/6178728)]



## 有向无环图的应用

### 拓扑排序

&#8195;&#8195;在一个表示工程的有向图中，有顶点表示活动，用弧表示活动之间的优先关系的有向图称为“顶点表示活动的网”，简称 AOV-网（Activity On Vertex Network）。AOV 就是用顶点表示活动，弧表示活动之间存在的某种制约关系。

&#8195;&#8195;在图论中，由一个有向无环图（directed acyclic graph，或称DAG）的顶点组成的序列，当且仅当满足下列条件时，称为该图的一个拓扑排序（Topological sorting）：

> 1. 每个顶点出现且只出现一次；
> 2. 若 A 在序列中排在 B 的前面，则在图中不存在从 B 到 A 的路径。



&#8195;&#8195;每一个有向无环图都至少存在一种拓扑排序。拓扑排序主要用来解决有向图中的依赖解析（dependency resolution）问题，举例来说，如果我们将一系列需要运行的任务构成一个有向图，图中的有向边则代表某一任务必须在另一个任务之前完成这一限制。那么运用拓扑排序，我们就能得到满足执行顺序限制条件的一系列任务所需执行的先后顺序。当然也有可能图中并不存在这样一个拓扑顺序，这时候我们无法根据给定要求完成这一系列任务，这种情况称为循环依赖（circular dependency）。

**算法思想：**

> 1. 在有向图中选一个没有前驱的顶点，输出；
> 2. 从图中删除该顶点和所有以它为尾的弧；
> 3. 重复直到所有顶点已输出 。



**算法步骤：**

> 1. 定义字典 indeg_num 存储各顶点的入度，定义列表 indeg_zero 存储入度为 0 的顶点，定义列表 tp 存储拓扑排序；
> 2. 从 indeg_zero 中取出一个顶点 v 放入 tp，已选定 v 后就把以 v 为起始的顶点 入度 - 1，此时如果入度为 0 则加入 indeg_zero ；
> 3. 重复上一步直到 indeg_zero 为空 。



```python
def topologica(graph):
    indeg_num = dict([(v, 0) for v in graph.keys()])
    # 初始化各顶点入度
    for vertexs in graph:
        for v in graph[vertexs]:
            indeg_num[v] += 1
    indeg_zero = [v for v in graph.keys() if indeg_num[v] == 0]
	print(indeg_num)
    tp = []
    # 输出入度为 0 的顶点 v，并使以 v 为起始的顶点 入度 - 1
    while indeg_zero:
        vertex = indeg_zero.pop()
        tp.append(vertex)
        for v in graph[vertex]:
            indeg_num[v] -= 1
            if indeg_num[v] == 0:
                indeg_zero.append(v)
    print(tp)

if __name__ == '__main__':
    graph = {'a': {'b', 'f'},
             'b': {'c', 'd', 'f'},
             'c': {'d'},
             'd': {'e', 'f'},
             'e': {'f'},
             'f': {}}

    topologica(graph)
    
# {'a': 0, 'b': 1, 'c': 1, 'd': 2, 'e': 1, 'f': 4}
# ['a', 'b', 'c', 'd', 'e', 'f']
```



[深入理解拓扑排序（Topological sort)](https://www.jianshu.com/p/3347f54a3187)



### 关键路径

&#8195;&#8195;拓扑排序是解决一个工程能否顺序进行的问题，但有时还需解决工程完成需要的最短时间。与  AOV-网对应的是 AOE-网（Activity On Edge Network），即“边表示活动的网”，**AOE-网是一个带权的有向无环图，用顶点表示事件，弧表示活动，权值表示活动的持续时间** 。通常 **AOE-网可用来估算工程完成的时间，AOV-网用来描述活动之间的制约关系**，和 AOV-网不同，对 AOE-网有待研究的问题是：

> 1.  完成整项工程至少需要多少时间？
> 2.  哪些活动是影响工程进度的关键？



&#8195;&#8195;如图 7.29 中有 9 个事件 11 项活动，$v_1$ 表示整个工程开始，$v_9$ 表示整个工程结束。只有活动 $a_4$ 和 $a_5$ 完成后事件 $v_5$ 才能发生，事件 $v_5$ 发生后活动 $a_7$ 和 $a_8$ 才可以开始。正常情况下网中只有一个入度为零的点（源点）和一个出度为零的点（汇点）。

![aoe](\imgs\aoe.png)



&#8195;&#8195;路径上各个活动所持续的时间之和称为路径长度，**从源点到汇点具有最大长度的路径就叫关键路径**，为什么是长度最大呢？只有最慢的活动完成后，整个工期才算完成，所以只有缩短关键路径上的活动才能缩短整个工程时间，一个工程可能有多条关键路径。

&#8195;&#8195;图中关键路径是 $(v_1. v_2. v_5, v_8, v_9)$，路径长度是 18。先看几个定义：

> 1. 事件最早开始时间 $ve(i)$：假设 $v_1$ 是起点，从 $v_1$ 到 $v_i$ 的长度就叫事件 $v_i$ 的最早开始时间，这个时间决定了所有以 $v_i$ 为尾的弧所表示的活动的最早开始时间，图中事件 $v_6$ 的最早开始时间是 7，这也是以它为尾的活动（即 $a_9$ ）的最早开始时间。
> 2. 事件最迟开始时间 $vl(i)$：事件 $v_i$ 最迟开始的时间，也是以它为尾的活动的最早开始时间，超出则会延误整个工期。
> 3. 活动最早开始时间 $e(i)$：活动 $a_i$ 最早开始的时间。图中活动 $a_6$ 最早开始时间是 5。
> 4. 活动最迟开始时间 $l(i)$：活动 $a_i$ 最迟开始的时间。在不推迟整个工程完成的前提下，活动最迟必须开始的时间，图中 $a_6$ 最迟开始时间是 18 - 4 - 4 - 2 = 8，也就是 $a_6$ 推迟 3 天开始或者延迟 3 天完成都不会导致整个工程延期。
> 5. 活动持续时间 $dut(<j, k>)$：如果活动 $a_i$ 由弧 $<j, k>$ 表示，则其持续时间记为 $dut(<j, k>)$



&#8195;找关键路径的过程就是辨别哪些是关键活动的过程，辨别关键活动就是要找 $e(i) = l(i)$ 的活动，也就是我们要找的这个活动，它的最早开始时间和最迟开始时间是相同的。如果活动 $a_i$ 由弧 $<j, k>$ 表示，则有如下关系：

> 1. $e(i) = ve(j)$ ，即活动 $a_i$ 的最早开始时间和事件 $v_j$ 的最早开始时间相同，如上图中 $a_6$ 的最早开始时间和事件 $v_4$ 的最早开始时间相同。
> 2. $l(i) = vl(k) - dut(<j, k>)$ ，即活动 $a_i$ 的最迟开始时间 =  事件 $v_k$ 的最迟开始时间 - 活动 $a_i$ 的持续时间（弧 $<j, k>$ 的长度），如上图中活动 $a_6$ 的最迟开始时间就是事件 $v_6$ 的最迟开始时间减去 $a_6$ 的持续时间。

&#8195;&#8195;一个活动的最早和最迟开始时间，其实就是这个活动起始事件的最早和最迟开始时间，为了求得 AOE-网中活动 $a_i$ 的 $e(i)$（活动最早开始时间）和 $l(i)$（活动最迟开始时间），首先应求得事件 $v_j$ 的 $ve(j)$（事件最早开始事件）和 $vl(j)$（事件最迟开始时间）， $ve(j)$ 可以利用 <font color = "red">拓扑排序从前往后</font> 求得，$vl(j)$ 可以利用 <font color = "red">拓扑逆序从后往前</font> 求得。

**算法步骤**
> 1. 对图进行拓扑排序，在拓扑排序的基础上求出各事件的最早开始时间，最早开始时间其实就是起点到该点最长的那条路径。
> 2. 初始化一个字典 $vl$ 用于存放事件最迟开始时间，初始化各事件的最迟开始时间均为汇点的最早开始时间，求最迟开始时间需要用拓扑逆序，其实就是把拓扑排序的列表当作栈，一个一个弹出顶点然后求最迟开始时间。
> 3. 根据各顶点的 $ve$ 和 $vl$，求每条弧 s 的最早开始时间 $e(s)$ 和最迟开始时间 $l(s)$，若某条弧满足 $e(s) = l(s)$，则为关键活动。


代码实现：
```python
class solution():
    # 拓扑排序，求顶点最早开始时间
    def topological(self, graph):
        indegree = dict([(v, 0) for v in graph.keys()])
        ve = dict([(v, 0) for v in graph.keys()])
        for vertexs in graph:
            for v in graph[vertexs]:
                indegree[v] += 1
        indeg_zero = [v for v in graph.keys() if indegree[v] == 0]
        #print(indegree)
        tp = []
        while indeg_zero:
            vertex = indeg_zero.pop()
            tp.append(vertex)
            for v in graph[vertex]:
                indegree[v] -= 1
                if indegree[v] == 0:
                    indeg_zero.append(v)
                if ve[vertex] + graph[vertex][v] > ve[v]:
                    ve[v] = ve[vertex] + graph[vertex][v]
        #print(tp, '\n', ve)
        if len(tp) < len(graph):
            print("cycle!!!")
            return False
        else:
            return tp, ve

    # 使用拓扑排序求关键路径
    def criticalpath(self, graph):
        tp, ve = self.topological(graph)
        vl = dict([(v, ve[tp[-1]]) for v in graph.keys()])
        print(tp, vl)
        # 拓扑逆序，求各顶点最迟开始时间
        while tp:
            vertex = tp.pop()
            for v in graph[vertex]:
                if vl[v] - graph[vertex][v] < vl[vertex]:
                    vl[vertex] = vl[v] - graph[vertex][v]
        
        # 找最早开始时间和最迟开始是将相等的弧
        for vertex in graph:
            for v in graph[vertex]:
                ee = ve[vertex]
                el = vl[v] - graph[vertex][v]
                if ee == el:
                    print(vertex, v)


if __name__ == '__main__':
    graph2 = {'v1': {'v2': 6, 'v3': 4, 'v4': 5},
              'v2': {'v5': 1},
              'v3': {'v5': 1},
              'v4': {'v6': 2},
              'v5': {'v7': 9, 'v8': 7},
              'v6': {'v8': 4},
              'v7': {'v9': 2},
              'v8': {'v9': 4},
              'v9': {}
              }

    s = solution()
    s.criticalpath(graph2)
```

&#8195;&#8195;如下表，顺序求出 $ve$ 逆序求出 $vl$ ：

|  | 事件最早开始时间 $ve$ | 事件最迟开始时间 $vl$ |
|--|--|--|
| <font color = "red">$v_1$</font> | <font color = "red">0</font>  | <font color = "red">0</font> (18 - $<v_1, v_9>$ = 18 - 18)|
| <font color = "red">$v_2$</font> | <font color = "red">6</font>  | <font color = "red">6</font> (18 - $<v_2, v_9>$ = 18 - 12)|
| $v_3$ | 4  | 6 (18 - $<v_3, v_9>$ = 18 - 12) |
| $v_4$ | 5  | 8 (18 - $<v_4, v_9>$ = 18 - 10)|
| <font color = "red">$v_5$</font> | <font color = "red">7</font>  | <font color = "red">7</font> (18 - $<v_5, v_9>$ = 18 - 11) |
| $v_6$ | 7  | 10(18 - $<v_6, v_9>$ = 18 - 8) |
| <font color = "red">$v_7$</font> | <font color = "red">16</font> | <font color = "red">16</font>(18 - $<v_7, v_9>$ = 18 - 2) |
| <font color = "red">$v_8$</font> | <font color = "red">14</font> | <font color = "red">14</font>(18 - $<v_8, v_9>$ = 18 - 4) |
| <font color = "red">$v_9$</font> | <font color = "red">18</font> | <font color = "red">18</font> |


&#8195;&#8195;有了各事件的最早和最迟开始时间，就很容易求出各活动的最早和最迟开始时间，最终得到 $<v_1, v_2>, <v_2, v_5>, <v_5, v_7>, <v_5, v_8>, <v_7, v_9>, <v_8, v_9>$ 。

&#8195;&#8195;由于拓扑排序必定在网中无环的前提下进行，则也可以利用 DFS 算法，在退出 DFS 之前计算顶点 $v$ 的 $vl$（因为此时 $v$ 的所有直接后继的 $vl$ 都已求出）。两种算法时间复杂度均为 $O(n + e)$ 。

## 并查集

&#8195;&#8195;并查集是一种树型的数据结构，用于处理一些不交集（Disjoint Sets）的合并及查询问题。有一个联合-查找算法（union-find algorithm）定义了两个用于此数据结构的操作：

> 1. Find：确定元素属于哪一个子集。它可以被用来确定两个元素是否属于同一子集。下面代码中我们定位为 `father()`。
> 2. Union：将两个子集合并成同一个集合。下面代码中我们定义为 `merge_node()`。


### 实现方法

&#8195;&#8195;并查集主要就两步，第一步查找两个元素是否属于一个集合（在图中的表现即，它们是否有共同的根节点），第二步如果它们不属于一个集合，就把它们合并成到同一集合（在图中的表现即，让一个元素的根节点成为另一个元素根节点的子节点）。

&#8195;&#8195;定义一个列表 `parent` ，其下标就是该元素的根节点，比如 `parent = [0, 0, 0, 2]` 代表 0, 1, 2 的根节点是 0 ，3 的根节点是 2 ，`father(self, x, parent)`  用于查询 x 的根节点， `merge_node(self, x, y, parent)` 用于合并 x 和 y元素，代码中的注释部分用于优化和调试，后面讲。

**代码实现：**

```python
class Solution:
    def union_find(self, edges, num):
        parent = [i for i in range(num)]
        #size = [1 for _ in range(num)] # 基于 size 的优化
        #rank = [1 for _ in range(num)] # 基于 rank 的优化
        for x, y in edges:
            x_root = self.father(x, parent)
            #x_root = self.father_iteration(x, parent) # 路径压缩优化
            #x_root = self.father_recursion(x, parent) # 路径压缩优化
            y_root = self.father(y, parent)
            #y_root = self.father_iteration(y, parent) # 路径压缩优化
            #y_root = self.father_recursion(y, parent) # 路径压缩优化
            if x_root == y_root:
                print("have a common root node:", x_root)
            else:
                self.merge_node(x_root, y_root, parent)
                #self.merge_node_size(x_root, y_root, parent, size)
                #self.merge_node_rank(x_root, y_root, parent, rank)

        print("parent:", parent)

    def father(self, x, parent):
        while x != parent[x]:
            x = parent[x]
        return x

    def merge_node(self, x, y, parent):
        parent[y] = x
        print("parent:", parent)
        #print("合并节点:", parent)


if __name__ == '__main__':
    vertex_num = 11
    test_edges = [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5],
                  [6, 7], [9, 10], [8, 10], [7, 9], [2, 8]]

    s = Solution()
    s.union_find(test_edges, vertex_num)

# parent: [0, 0, 0, 0, 0, 0, 0, 6, 6, 8, 9]
```

&#8195;&#8195;结果如下右图：

![union-find-common](\imgs\union-find-common.png)



### 基于 Union 的优化

&#8195;&#8195;就是对合并过程进行优化。在上面的算法中， `merge_node(self, x, y, parent)` 只是固执的把 y 作为 x 的子节点，在一些极端情况下会产生单链表格式的结构，导致在做 find 查询时效率会大打折扣，特别在数据量较大的时候，这样的劣势体现得更加的明显，所以我们需要在合并元素时候做一些优化。



**基于 size 的优化：**

&#8195;&#8195;我们可以引入一个列表 `size`，专门用来记录每一个集合都有多少个元素，然后在进行合并时候，先查询 `size` 中两个集合的元素个数，把元素个数小的集合并入元素个数大的集合中。这样，就能大大的减少因为合并造成树的层数过高的现象，提高 find 效率。 `size` 初始化每个元素各自为一个集合，个数均为 1 ，代码如下：

```python
    def merge_node_size(self, x, y, parent, size):
        if size[x] < size[y]:
            parent[x] = y
            size[y] += 1
        else:
            parent[y] = x
            size[x] += 1
        print("合并节点:", parent, size)
        

# parent: [0, 0, 0, 0, 0, 0, 9, 6, 9, 0, 9]
```

&#8195;&#8195;优化后如下左图：

![union-find-size-rank](\imgs\union-find-size-rank.png)



&#8195;&#8195;基于 size 的优化，通过判断两个集合元素的数量大小来决定把哪一个集合并入另一个集合当中，从而降低了树高，使得 find 操作的效率更高。但这中优化似乎也有局限性，如果一个集合 A 元素很多却只有两层，而另一个集合 B 元素很少却有三层，这时候明显把 A 并到 B 更优，总层数就是 B 的层数，但是把 B 并入 A 的话总层数会比 B 的层数还高 1 。

&#8195;&#8195;在集合中，层数越少，对于每一个节点平均来说，找到根节点所需要查找的次数就会越小，所以接下来我们看看对层数的优化。



**基于 rank 的优化：**

&#8195;&#8195;我们可以引入一个列表 `rank`，专门用来记录两个集合的层数，然后在进行合并时候，先查询 `rank` 中两个集合的层数，层数小的集合并入层数大的集合中。这样，就能更准确的降低合并过程中树的层数，提高 find 效率。 `rank` 初始每个元素各自是一个集合，层数均为 1 ，代码如下：

```python
    def merge_node_rank(self, x, y, parent, rank):
        if rank[x] < rank[y]:
            parent[x] = y
        elif rank[x] > rank[y]:
            parent[y] = x
        else:
            parent[y] = x
            rank[x] += 1
        print("合并节点:", parent)

# parent: [6, 0, 0, 0, 0, 0, 6, 6, 9, 6, 9]
```

&#8195;&#8195;合并后集合层数唯一会变的情况，就是两个集合层数一模一样的时候。层数相同时，谁是谁的根节点都无所谓了。这两种优化方式其在非极端情况下其实差不多，不过基于 rank 的优化更能应对极端情况。结果如上右图。



### 基于 Find 的优化

&#8195;&#8195;上面都是对合并过程的优化，现在我们对查询过程进行优化，也就是路劲压缩优化。可不可以在 find 的同时，做一些操作使得树的层数尽量变得更少呢？

&#8195;&#8195;对于一个集合树来说，它的根节点下面可以依附着许多的节点，因此，我们可以尝试在 find 的过程中，从底向上，如果此时访问的节点不是根节点的话，那么我们可以把这个节点尽量的往上挪一挪，减少集合的层数，这个过程就叫做路径压



**基于循环的路径压缩优化**

&#8195;&#8195;假设我们起始的并查集如下图所示，现在我们要 find[4] ，首先我们根据 parent[4] 可以得出 4 并不是一个根节点，因此我们可以在向上继续查询之前，先把这个节点往上面挪一挪（路径压缩），我们让节点 4 连接到其父节点的父节点上，让节点 2作为其新的父亲节点，如果节点 2 正好是根节点，那么 2 的父节点指向自己，所以这种变动并不会因为越界而出错。

![union-path1](\imgs\union-path1.png)

&#8195;&#8195;在进行 find 操作的同时，我们不仅把需要查找的根节点给找到了，与此同时我们还对树进行了压缩操作，这便是路径压缩的意思。通过路径压缩，我们在下一次执行 find 操作的时候，层数变得尽可能少了，那么效率将会大大的提高。

代码如下：

```python
    def father_iteration (self, x, parent):
        while x != parent[x]:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x
    
# parent: [6, 0, 0, 0, 0, 0, 6, 6, 6, 6, 9]
```

&#8195;&#8195;查询节点时候，顺便让该节点指向了其父节点的父节点，也就是大于等于 3 层的结构中才能体现出其优势。其实这种路径压缩方法并没有把层数压缩到最小，最优情况下应该压缩到两层，所有的节点都指向根节点，这就需要使用基于递归的路径压缩。



**基于递归的路径压缩优化**

&#8195;&#8195;最优情况下应该压缩到两层，如下图：



![union-path2](\imgs\union-path2.png)



&#8195;&#8195;因为 find 操作返回的是集合的根节点，因此我们只需要集合中所有的非根节点的父亲指针都指向这个根节点就好了，我们完全可以用递归的方法去实现，递归是典型的用空间换时间的方法。

代码如下：

```python
    def father_recursion(self, x, parent):
        if x == parent[x]:
            return x
        else:
            parent[x] = self.father_recursion(parent[x], parent)
            return parent[x]
        
# parent: [6, 0, 0, 0, 0, 0, 6, 6, 6, 6, 9]        
```

&#8195;&#8195;用上面的测试用例，这两种情况效果一样，我们需要用图中结构的测试用例才行，代码逻辑很好理解，不再测试。

&#8195;&#8195;代码逻辑：

> 1. 首先设终止条件及其操作，如果访问的节点是根节点，则直接返回。
> 2. 非终止条件下，`father_recursion(parent[x], parent)` 使得递归往根节点深入下去，并且把找到的根元素赋值给节点的父亲指针。`father_recursion(parent[x], parent)` 返回来的是集合中的根元素，我们把现在正在访问的这个元素的父亲指针指向上一步返回回来的 `father_recursion(parent[x], parent)` ，此时的 `parent[x]` 就不仅仅是现在访问的元素的父亲节点了，还是集合树中的根节点，我们在把 `parent[x]` 返回给再上面的一层，让其元素的父亲指针也指向 `parent[x] `（即根节点），这样一层接一层，最后这条递归传递路线上所有的元素的父亲指针都会指向那个根节点了。
> 3. `return parent[x]` 返回此时访问的节点的父亲指针指向的节点，也就是这个集合的根节点。



### 最终代码

```python

class Solution:
    def union_find(self, edges, num):
        parent = [i for i in range(num)]
        rank = [1 for _ in range(num)]
        for x, y in edges:
            x_root = self.father_iteration(x, parent)
            y_root = self.father_iteration(y, parent)
            if x_root == y_root:
                print("have a common root node:", x_root)
            else:
                self.merge_node_rank(x_root, y_root, parent, rank)
        print("parent:", parent)

    def father_iteration (self, x, parent):
        while x != parent[x]:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def merge_node_rank(self, x, y, parent, rank):
        if rank[x] < rank[y]:
            parent[x] = y
        elif rank[x] > rank[y]:
            parent[y] = x
        else:
            parent[y] = x
            rank[x] += 1
        print("合并节点:", parent)

if __name__ == '__main__':
    vertex_num = 11
    test_edges = [[0, 1], [1, 2], [2, 3], [3, 4], [4, 5],
                  [6, 7], [9, 10], [8, 10], [7, 9], [2, 8]]

    s = Solution()
    s.union_find(test_edges, vertex_num)

```



[并查集的常规实现方法](<https://blog.csdn.net/qq_19782019/article/details/78909801>)

[基于Size的并查集优化](<https://blog.csdn.net/qq_19782019/article/details/78914328>)

[基于Rank的并查集优化](<https://blog.csdn.net/qq_19782019/article/details/78916595>)

[并查集的路径压缩优化](<https://blog.csdn.net/qq_19782019/article/details/78919990>)
