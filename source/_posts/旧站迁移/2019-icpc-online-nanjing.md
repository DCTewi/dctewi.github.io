---
title: '【总结】2019南京网络赛小结'
date: 2019-09-06 11:45:12
tags: [题解,动态规划,最短路,树状数组,欧拉定理,离散化]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kaf-hand-light.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---

The Preliminary Contest for ICPC Asia Nanjing 2019 （即 ICPC2019 南京网络赛）部分总结

<!-- more -->

## A - The beautiful values of the palace

### 题目大意

整块 n * n 的地区，每个坐标都有自己的权值。权数值由中间到外围螺旋状减小（如下例），权值等于权数值的**数位**总和。在这片地区有 m 座城堡，对于 q 次询问，询问一个矩形范围内的城堡占点的权值和。

对于 n = 3 和 n = 5 的情况，点权分布情况为：

```xml
7 8 1           13 14 15 16 01
6 9 2           12 23 24 17 02
5 4 3           11 22 25 18 03
                10 21 20 19 04
                09 08 07 06 05
```

其中，左下角为`(1, 1)`，右上角为`(n, n)`。T 组数据。

数据范围：`T <= 5; n <= 1e6; m, p <= 1e5`。

### 题目思路

将数据离散化后仍然需要 1e5 * 1e5 的大小来存储数据，不可行。

将二维问题化为一维问题，将所有数据存储到同一个结构体中，按 x 轴排序后根据 y 轴大小维护一个树状数组。

其中地图信息和询问信息全部离线到数组中统一处理，对于坐标重合的数据，将地图信息排在询问的前边。最后扫描统一处理。

### 参考写法

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MAXN = 1e6 + 5;

struct Node // 存储信息的结构体
{
    int x, y; // 坐标
    int mark, id; // 标记，数值

    bool operator< (const Node &o) const
    {
        return (x != o.x)? x < o.x:
            (y != o.y)? y < o.y:
            mark > o.mark;
    }
} tile[MAXN] ;
int num; // 离线的信息总大小
void addtile(int X, int Y, int M, int I)
{
    num++;
    tile[num].x = X;
    tile[num].y = Y;
    tile[num].mark = M;
    tile[num].id = I;
}

// 树状数组
ll c[MAXN], top;
inline int lowbit(int x)
{
    return x & (-x);
}
void add(int pos, int val)
{
    while (pos <= top)
    {
        c[pos] += val;
        pos += lowbit(pos);
    }
}
ll query(int pos)
{
    ll res = 0;
    while (pos > 0)
    {
        res += c[pos];
        pos -= lowbit(pos);
    }
    return res;
}

// 计算 (x, y) 在一个长宽为 len 的地图中的权值
int getxy(int x, int y, int len)
{
    int wid = min(min(x - 1, y - 1), min(len - x, len - y)); // 差值
    ll val = 2ll * (2 * len - 2 - 2 * (wid - 1)) * wid; // 数值
    len -= wid * 2;
    x -= wid; y -= wid;

    if (x == len && y > 1) val += (len - y + 1);
    else if (y == 1 && x > 1) val += (len - 1 + len - x + 1);
    else if (x == 1 && y < len) val += (2 * (len - 1) + y);
    else val += (3 * (len - 1) + x);
    
    int res = 0; // 分解数值
    while (val)
    {
        res += val % 10;
        val /= 10;
    }
    return res;
}

// 模板
template <class T> T read()
{
    T x = 0; int w = 0, ch = getchar();
    while (!isdigit(ch)) w |= ch == '-', ch = getchar();
    while (isdigit(ch)) x = (x << 3) + (x << 1) + (ch ^ 48), ch = getchar();
    return w? -x: x;
}

int n, m, q;
ll ans[MAXN];

int main()
{
    int T = read<int>();
    while (T--)
    {
        n = read<int>(), m = read<int>(), q = read<int>();
        // 初始化
        for (int i = 0; i <= n; i++) c[i] = 0;
        memset(ans, 0, sizeof(ans)); 
        top = n;
        num = 0;
        // 读入地图数据，标记类型为 2
        for (int i = 1; i <= m; i++)
        {
            int a = read<int>(), b = read<int>();
            addtile(a, b, 2, getxy(a, b, n));
        }

        for (int i = 1; i <= q; i++)
        {
            int a = read<int>(), b = read<int>(), c = read<int>(), d = read<int>();            
            addtile(c, d, 1, i); // 进入询问区间，加上相关大小
            addtile(a - 1, b - 1, 1, i); 
            addtile(a - 1, d, -1, i); // 离开询问区间，还原
            addtile(c, b - 1, -1, i);
        }

        sort(tile + 1, tile + 1 + num); // 排序

        // 扫描处理问题
        for (int i = 1; i <= num; i++)
        {
            if (tile[i].mark == 2) add(tile[i].y, tile[i].id); // 城堡进入
            else
            {
                ans[tile[i].id] += query(tile[i].y) * tile[i].mark; // 处理
            }
        }
        for (int i = 1; i <= q; i++)
        {
            printf("%I64d\n", ans[i]); // 输出
        }
    }
    return 0;
}

```

## B - super_log

### 题目大意

求 a 的 b 个 a 次幂模 m 的值，即： 
$$
\begin{equation}
\underbrace{a^{a^{a^{...}}}}_{b\ times} (mod\ m)
\end{equation}
$$

### 题目思路

使用欧拉定理递归降幂，需要特判 a 和 m 不互质的情况。

欧拉函数会很快迭代到 1，可以提前退出，速度很快。

扩展欧拉定理：
<math xmlns="http://www.w3.org/1998/Math/MathML" display="block"><msup><mi>a</mi><mi>b</mi></msup><mo>≡</mo><mrow data-mjx-texclass="INNER"><mo data-mjx-texclass="OPEN">{</mo><mtable columnalign="left left" columnspacing="1em" rowspacing=".2em"><mtr><mtd><msup><mi>a</mi><mi>b</mi></msup></mtd><mtd><mi>g</mi><mi>c</mi><mi>d</mi><mo stretchy="false">(</mo><mi>a</mi><mo>,</mo><mi>p</mi><mo>≠</mo><mn>1</mn><mo>,</mo><mi>b</mi><mo>&lt;</mo><mi>ϕ</mi><mo stretchy="false">(</mo><mi>p</mi><mo stretchy="false">)</mo><mo stretchy="false">)</mo></mtd></mtr><mtr><mtd><msup><mi>a</mi><mrow><mi>b</mi><mi mathvariant="normal">%</mi><mi>ϕ</mi><mo stretchy="false">(</mo><mi>p</mi><mo stretchy="false">)</mo><mo>+</mo><mi>ϕ</mi><mo stretchy="false">(</mo><mi>p</mi><mo stretchy="false">)</mo></mrow></msup></mtd></mtr></mtable><mo data-mjx-texclass="CLOSE" fence="true" stretchy="true" symmetric="true"></mo></mrow><mtext>&nbsp;</mtext><mtext>&nbsp;</mtext><mo stretchy="false">(</mo><mi>m</mi><mi>o</mi><mi>d</mi><mtext>&nbsp;</mtext><mi>p</mi><mo stretchy="false">)</mo></math>

## F - Greedy Sequence

### 题目大意

对于 n 个数字，k 的搜索半径，一个长度为 n 的字典池。要求以 1 到 n 为序列首位，构造 n 个序列。

对于每个序列，每次以当前最后一位为原点，选择字典池中半径为 k 的最大值，直到无法选择。

如对于 `n = 7, k = 2, dic[] = {3, 1, 4, 6, 2, 5, 7}`，构造的七个序列分别为：

```xml
1
2
3, 1
4, 3, 1
5, 2
6, 5, 2
7, 5, 2
```

求构造的 n 个序列的长度，如上述样例，输出为`1 1 2 3 2 3 3 `。

### 题目思路

简单的 DP，字典序最大，则每次只选择能选择的最大值，找到后 break 即可。

### 参考写法

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MAXN = 1e5 + 5;

int n, k;
map<int, int> plc; // 每个字符的位置

template <class T> T read()
{
    T x = 0; int w = 0, ch = getchar();
    while (!isdigit(ch)) w |= ch == '-', ch = getchar();
    while (isdigit(ch)) x = (x << 3) + (x << 1) + (ch ^ 48), ch = getchar();
    return w? -x: x;
}

int main()
{
    int T = read<int>();
    while (T--)
    {
        n = read<int>(), k = read<int>();
        vector<int> dp(n + 1, 1); // 初始化，每个序列最小长度为 1

        for (int i = 0; i < n; i++)
        {
            int u = read<int>();
            plc[u] = i;
        }

        dp[1] = 1; // 以 1 开头长度一定为 1
        for (int i = 2; i <= n; i++)
        {            
            for (int j = i - 1; j > 0; j--)
            {
                int len = abs(plc[j] - plc[i]);
                if (len > k) continue;

                dp[i] = dp[j] + 1; break;
            }
        }

        for (int i = 1; i <= n; i++)
        {
            printf("%d%c", dp[i], " \n"[i == n]);
        }
    }
}

```

## H - Holy Grail

### 题目大意

你的英灵的宝具是一个带负权有向图（？）。你需要在指定的六个位置依次连接六条此位置允许的最小边，使不出现负权环，否则你会被圣杯吞噬（？）。数据保证答案存在。

### 题目思路

要使不出现负权环，则需要在指定位置创建零权环。要在 x -> y 连边创建零权环，则所需权值为 y -> x 最短路的相反数。每次跑一下最短路并连接新边即可。

### 参考写法

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

const int MAXN = 305;
const int MAXM = 505;

template <class T> T read()
{
    T x = 0; int w = 0, ch = getchar();
    while (!isdigit(ch)) w |= ch == '-', ch = getchar();
    while (isdigit(ch)) x = (x << 3) + (x << 1) + (ch ^ 48), ch = getchar();
    return w? -x: x;
}

struct Edge // 链式前向星
{
    int to;
    ll cost;
    int next;
    
    Edge(int t = 0, ll c = 0, int ne = 0)
    {
        to = t; cost = c; next = ne;
    }
};

struct Spfa // SPFA
{
    vector<int> head;
    queue<int> q;
    vector<Edge> edges;
    ll dis[MAXN];
    int vis[MAXN];

    Spfa(int maxn)
    { // 构造时初始化
        head = vector<int>(maxn, 0);
        edges = vector<Edge>(1);
    }

    void addedge(int u, int v, ll w)
    {
        edges.emplace_back(Edge(v, w, head[u]));
        head[u] = edges.size() - 1;
    }

    ll run(int start, int fin) // 开跑
    {
        memset(vis, 0, sizeof(vis));
        memset(dis, 0x3f, sizeof(dis));

        dis[start] = 0; vis[start] = 1; q.push(start);

        while (q.size())
        {
            int u = q.front();
            for (int i = head[u]; i; i = edges[i].next)
            {
                int v = edges[i].to, w = edges[i].cost;
                if (dis[v] > dis[u] + w)
                {
                    dis[v] = dis[u] + w;
                    if (!vis[v])
                    {
                        q.push(v); vis[v] = 1;
                    }
                }
            }
            q.pop(); vis[u] = 0;
        }
        
        return dis[fin];
    }
};

int main()
{
    int T = read<int>();
    while (T--)
    {
        int n = read<int>(), m = read<int>();
        Spfa spfa(n);

        for (int i = 0; i < m; i++)
        {
            int u = read<int>(), v = read<int>(); ll w = read<ll>();
            spfa.addedge(u, v, w);
        }

        for (int i = 0; i < 6; i++)
        {
            int x = read<int>(), y = read<int>();

            ll now = spfa.run(y, x); // 反向最短路

            spfa.addedge(x, y, -now); // 连边
            printf("%I64d\n", -now); // 输出
        }
    }
    return 0;
}

```

