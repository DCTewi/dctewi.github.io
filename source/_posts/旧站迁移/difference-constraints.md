---
title: '【算法】差分约束小结'
date: 2019-07-19 17:00:01
tags: [差分约束,SPFA]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
差分约束系统(System of Difference Constraints)的基本概念，思路和用法以及例题总结。

<!-- more -->

## 基本概念

**差分约束系统**(System of Difference Constraints)，是求解关于一组变数的特殊不等式组的方法。

如果一个系统由n个变量和m个约束条件组成，其中每个约束条件形如 $x_i-x_j<=b_k (i, j\in[1, n],\ k\in[1, m])$，则称其为差分约束系统。

## 代数解法

求差分约束的最大值，可以将某几个不等式加减组合形成我们需要求解的不等式，然后得到答案。

## 图论解法

差分约束系统都可以转化成单源最短路（或者最长路）问题。

**转化：**

将 $x[i]-x[j]<=b[k]$ 移项转化为 $x[i]<=x[j]+b[k]$ ，形式上与松弛操作`dis[v] = dis[u] + edge[u][v]`相似。

则我们将每个不等式转化成一条边，`x[i] - x[j] <= b[k]`转化为由 j 到 i ，边权为 b[k] 的有向边。

则要求`x[n - 1] - x[0]`的最大值，可以转化为求 `0 -> n - 1`的最短路；

反之，最小值将转化为两点之间的最长路问题。

## 一种理解方式

求`x[i] - x[j] <= b[k]`的最大值，则需要求这些`b[k]`中最小的，即约束最严苛的一个不等式。

反之，求`x[i] - x[j] >= b[k]`的最小值，则需要求这些`b[k]`中最大的值。

## 解的存在性判断

如同最短路问题中的负环 / 终点不联通情况，在求解差分约束问题时同样存在对应版本的特殊情况。

### 1 - 最大值负环 / 最小值正环

若求解最短路负环，则最短路可以是无限小。此时我们认为最大值和最短路不存在。

**对应的SPFA判断操作：**若某一点的入队次数大于节点数，则认为不存在。

### 2 - 终点未联通

表示两点之间没有约束关系，最大值和最小值为正负无穷。实现过程中由`dis[n - 1] = INF`来表示。

## 例题

- **题目链接：**[洛谷P1993 - 小K的农场](https://www.luogu.org/problemnew/show/P1993)

- **题意转化：**

  题目中给了三种情况，分别是：

  1. `a - b <= c`
  2. `a - b >= c`
  3. `a == b`

  分别转换成 `a >= b + c` 和  `b >= a - c`。

  要求判断是否条件不矛盾，当条件有环时则说明矛盾（如 `a > b, b > c, c > a`），则题目可以转化为判断是否有环。对于以上的形式：

  1. 建立` b --> a, cost = c`的边
  2. 建立`a --> b, cost = -c`的边
  3. 建立`a <-> b, cost = 0`的边

  对于大于等于，求最小值，最长路，则求是否存在正环（若整理成小于等于则恰好相反）。

- **优化点**

  对于这道题，我们只需要判断是否有环，所以我们可以采用DFS形式的SPFA。深度优先使得算法在查找环时的效率倍增。但是对于求距离的题目，则传统BFS更优。

  超级源点 0，通过边权为0的边连通每一个点，使图联通。

  链式前向星存图。

- **示例代码**

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 1e4 + 5;
const int INF = 1 << 30;
int n, m;

struct Edge
{
    int to, cost, next;
} edge[MAXN << 2];
int head[MAXN << 2];
void add_edge(int u, int v, int w)
{
    static int cnt = 0;
    cnt++;
    edge[cnt].to = v;
    edge[cnt].cost = w;
    edge[cnt].next = head[u];
    head[u] = cnt;
}

int vis[MAXN], dis[MAXN], flag = 0;
void spfa(int u)
{
    if (flag) return;
    vis[u] = 1;
    for (int i = head[u]; i; i = edge[i].next)
    {
        int v = edge[i].to, w = edge[i].cost;
        if (dis[v] < dis[u] + w)
        {
            if (vis[v] || flag)
            {
                flag = 1; break;
            }
            dis[v] = dis[u] + w;
            spfa(v);
        }
    }
    vis[u] = 0;
}

int main()
{
    ios::sync_with_stdio(0);
    cin >> n >> m;
    for (int i = 0; i < m; i++)
    {
        int opt, a, b, c;
        cin >> opt >> a >> b;
        switch (opt)
        {
            case 1:
            {
                cin >> c; add_edge(b, a, c);
                break;
            }
            case 2:
            {
                cin >> c; add_edge(a, b, -c);
                break;
            }
            case 3:
            {
                add_edge(a, b, 0); add_edge(b, a, 0);
                break;
            }
            default: break;
        }
    }

    for (int i = 1; i <= n; i++)
    {
        add_edge(0, i, 0); dis[i] = -INF;
    }

    spfa(0); 
    cout << ((flag)? "No\n": "Yes\n");
    return 0;
}

```

## 小结

以上就是差分约束的基本总结以及例题。

冻葱Tewi 于 2019-07-19