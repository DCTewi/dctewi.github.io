---
title: '【总结】A*搜索算法小结'
date: 2020-03-13 20:41:09
tags: [笔记,算法,A*,最短路]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kano-smile-p1.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---

A* 搜索是一种使用了 Dijkstra 思想的启发式搜索，是一种在图形平面上，有多个节点的路径，求出最低通过成本的算法。把之前进行的总结整理了一下发了上来。

<!-- more -->

## 启发式搜索

启发式搜索是搜索的主要优化方法之一，另一种常见的优化方法是记忆化搜索。

启发式搜索是一种在搜索的过程中，通过自定义估价函数$f(x)$对不同的子状态进行筛选、剪枝，从中选取更优解或者删去无效解的在线剪枝搜索。

**例题：[Luogu 1048 采药](https://www.luogu.com.cn/problem/P1048)**

当然，这道题是一道DP题，也是一道记忆化的题目。但是这道题也可以写成启发式搜索的形式：在取的时候判断是否超过了可行体积（可行性剪枝），不取的时候判断是否剩余药品的价值+当前价值>当前解（最优性剪枝）。

上代码：

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1e2 + 5;
int n, m, ans;
struct Node
{
    int time, value;
    double cost;

    bool operator<(const Node &o) const
    {
        return cost > o.cost;
    }
} nodes[MAXN];
// 估价函数：后方价值和
int f(int t, int v)
{
    int res = 0;
    for (int i = 1; i + t <= n; i++)
    {
        if (v >= nodes[t + i].time)
        {
            v -= nodes[t + i].time;
            res += nodes[t + i].value;
        }
        else return (int)(res + v * nodes[t + i].cost);
    }
    return res;
}
// 启发式搜索的动态剪枝
void dfs(int t, int p, int v)
{
    ans = max(ans, v);
    if (t > n) return;
    // 不选择
    if (f(t, p) + v > ans) dfs(t + 1, p, v);
    // 选择
    if (nodes[t].time <= p) dfs(t + 1, p - nodes[t].time, v + nodes[t].value);
}

int main()
{
    scanf("%d%d", &m, &n);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d%d", &nodes[i].time, &nodes[i].value);
        nodes[i].cost = 1.0 * nodes[i].value / nodes[i].time;
    }
    sort(nodes + 1, nodes + n + 1);
    dfs(1, m, 0);
    printf("%d\n", ans);
    return 0;
}
```

## A*

A Star 算法是 Dijkstra 和启发式搜索的结合。他的估价函数由两部分组成：$f(x)=g(x)+h(x)$，其中$𝑔(𝑛)$表示从起点到任意顶点 $𝑛$ 的实际距离，$ℎ(𝑛)$ 表示任意顶点 $𝑛$ 到目标顶点的估算距离（根据所采用的评估函数的不同而变化）。

整体算法过程：

1. Def Priority_Queue $Queue$
2. Do​
   1. Let​ $u$ => $u \in Queue$ && $f(u) = min\{f(x)\ |\ x\in Queue\}$
   2. For Each $v$ => $Exits(u\to v)$
      1. If $v \notin Queue$ Then $Queue.Push(v)$, $Update(f(v))$
      2. Else If $Cost(u\to v) < g(v)$ Then $Update(f(v))$
3. Until $Not\ Queue.Empty()$ || $Target\in Queue$

他的整体过程可以概括为：每次从优先队 中取出一个$𝑓(𝑖)$最小的状态，然后更新可用的相邻状态，最大复杂度$O(n^2)$，优先队列优化后的期望复杂度为$O(nlogn)$。

因为启发式动态剪枝的存在，所以相对于一般的寻路算法，期望的搜索范围小。并且 A* 在$h(x)$满足三角形不等式时，不会将重复节点加入队列。

### 竞赛中的A*

竞赛中的 A* 一般用来解决定点k短路的问题。即给出图，求两点之间第k条简单路。

我们预处理反向图中从终点出发的单源最短路作为$g(x)$，将从开始到当前状态走过的所有路径作为$h(x)$跑 A*。当我们第k次将终点入栈时，我们就找到了从起点到终点的第k短路。

**例题：[POJ 2449 Remmarguts' Date](http://poj.org/problem?id=2449)**

模板题，上代码：

```cpp
#include <bits/stdc++.h> // POJ啥时候支持C++11啊我佛了
using namespace std;
typedef long long ll;
const int MAXN = 1e3 + 5;
const int MAXM = 1e5 + 5;
const int MAXK = 1e3 + 5;
const int INF = 1 << 30;
// 链式前向星, 正图反图两个头数组，边都在一个数组里
struct Edge
{
    int v, w, next;
    Edge(int v = 0, int w = 0, int next = 0) : v(v), w(w), next(next) { }
} edge[MAXM << 2];
int headf[MAXN], headb[MAXN], edgecnt = 0; 
void addedge(int u, int v, int w)
{
    edge[++edgecnt] = Edge(v, w, headf[u]); headf[u] = edgecnt; // 正
    edge[++edgecnt] = Edge(u, w, headb[v]); headb[v] = edgecnt; // 反
}
int dis[MAXN], vis[MAXN], ans[MAXK]; // 反向最短路, 入队标记, 第i短路
int n, m, s, t, k; // 见题意, n点m边从s到t的第k短路
// 状态
struct State
{
    int u, cost, sum;
    bool operator<(const State &o) const
    {
        return sum == o.sum ? cost > o.cost : sum > o.sum;
    }
    State(int u = 0, int g = 0) : u(u), cost(g)
    {
        sum = g + dis[u];
    }
};
// A*本体
void astar()
{
    if (dis[s] == INF) return;
    if (s == t) k++;
    int cnt = 0;

    priority_queue<State> q;
    q.push(State(s, 0));
    while (q.size())
    {
        State e = q.top(); q.pop();
        if (e.u == t) // 检查抵达终点次数
        {
            ans[++cnt] = e.cost;
            if (cnt == k) return;
        }
        for (int i = headf[e.u]; i; i = edge[i].next)
        {
            q.push(State(edge[i].v, e.cost + edge[i].w));
        }
    }
}
// SPFA求反图最短路
void spfa()
{
    queue<int> q;
    q.push(t); vis[t] = 1; dis[t] = 0;
    while (q.size())
    {
        int u = q.front(); q.pop();
        for (int i = headb[u]; i; i = edge[i].next)
        {
            int v = edge[i].v;
            if (dis[v] > dis[u] + edge[i].w)
            {
                dis[v] = dis[u] + edge[i].w;
                if (!vis[v])
                {
                    q.push(v); vis[v] = 1;
                }
            }
        }
        vis[u] = 0;
    }
}
void init()
{
    memset(headb, 0, sizeof headb);
    memset(headf, 0, sizeof headf);
    memset(ans, -1, sizeof ans);
    memset(vis, 0, sizeof vis);
    for (int i = 0; i <= n; i++) dis[i] = INF;
    edgecnt = 0;
}
int main()
{
    while (~scanf("%d%d", &n, &m))
    {
        init();
        for (int i = 0; i < m; i++)
        {
            int u, v, w;
            scanf("%d%d%d", &u, &v, &w);
            addedge(u, v, w);
        }
        scanf("%d%d%d", &s, &t, &k);
        spfa();
        astar();
        printf("%d\n", ans[k]);
    }
}
```

### 开发中的A*

A*因为有着效率高，路径权值高度可定义化等特点，在游戏开发时经常配合Navmesh等技术用作AI寻路。比如有名的 [A Star Pathfinding Project](https://arongranberg.com/astar/) 。

## 最后

这次的内容比较简单，所以也没多少字可以~~水~~写，就只稍微总结了一些模板和代码。当然如果有所帮助就更好了。