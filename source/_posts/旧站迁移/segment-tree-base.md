---
title: '【数据结构】线段树基础小结'
date: 2019-02-11 23:02:23
tags: [数据结构,旧文归档,线段树]
cover: https://s2.ax1x.com/2019/02/11/kdi1Yt.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
学习两种线段树后的总结。
  可能并不适合完全不了解线段树的人入门？但是可能会对稍微知道线段树的人有一些帮助。

<!-- more -->

> **线段树**是一种二叉树，可以视为树状数组的变种。
>
> 该数据结构在实际应用中用途不大，但是由于其程序易于实现而呗广泛应用于程序竞赛当中。其用途是在O(logN)查询一个指定区间内的信息，并可在同样的时间复杂度内支持更新等操作。
>
> ——Wikipedia

### 基本概念

线段树本身是专门用来处理**区间问题**的数据结构，包括但不限于区间求和问题(RSQ)，区间最值问题(RMQ)等。因为本身具有树的结构特性而被称为线段“树”。

线段树是分块思想的树化，或者说是对于信息处理的二进制化。即——通过将整个序列分为有穷个小块，对于要查询的某一段区间，总是可以整合成k个**子块**和m个**单个元素**的信息的并集。

线段树只能维护满足结合律的信息，即只能维护一个幺半群序列（群论可以参见[群论与快速幂](https://dctewi.com/quick_exp/)）。而树状数组只能维护交换群序列。

线段树上的每一个子节点，都表示整个序列中的一段子区间。其中，每个叶子节点都表示序列中的单个元素信息。每次更新，子节点都向上传递信息，来不断更新整个线段树。

### 存储结构

以这一棵总值线段树为例：

![线段树示例](https://s2.ax1x.com/2019/02/11/kdPhz8.png)

树上每一个节点存储的都是一个区间上的总和。

## 常规递归线段树

### 建树操作

一般地，我们经常采用递归的方法来创建线段树。对于节点类：

```cpp
typedef long long ll;
const int MAXN = 1e5 + 5;  //源数据最大长度
class SegmentTreeNode
{
public:
    SegmentTreeNode(){}  //构造函数
    
    int l, r;  //该节点储存的区间左右范围
    ll data, add_tag = 0, mul_tag = 1;  //该节点应该储存的数据和标记*
}st[MAXN << 2];  //线段树数组，范围×4防止越界
ll raw_data[MAXN];  //原始数据

```

我们递归地从下向上来创建线段树：

```cpp
void build(int p, int l, int r)  //p节点序号，l, r范围
{
    st[p].l = l; st[p].r = r;  //处理当前区间

    if (l == r)  //叶节点特判
    {
        st[p].data = raw_data[l];  //赋值
        return ;  //触底返回
    }

    int mid = (l + r) >> 1;  //二分递归
    build(lson(p), l, mid);
    build(rson(p), mid + 1, r);

    push_up(p);  //上推*
}

```

### push_up()操作

push_up()即线段树上推操作，即把下方节点的更新情况"告知"上方节点。对于不同类型的线段树可能有所不同。在总值线段树中，上推操作为：

```cpp
void push_up(int p)
{
    st[p].data = (st[lson(p)].data + st[rson(p)].data) % mod;
}

```

### 懒惰标记

当我们更新时，如果每次都把整个线段树更新，那么将会耗费很多多余的时间。所以我们将每次节点的更新记录到节点内，当需要使用它下方的节点时在进行下推。标记记录的就是以这个节点为根节点的子树中经过过的更新操作。

下推懒惰标记的操作为：

```cpp
void lazy_down(int p)
{
    if (st[p].add_tag || st[p].mul_tag != 1)
    {
        st[lson(p)].data = st[lson(p)].data * st[p].mul_tag % mod;
        st[lson(p)].mul_tag = st[lson(p)].mul_tag * st[p].mul_tag % mod;
        st[lson(p)].add_tag = st[lson(p)].add_tag * st[p].mul_tag % mod;

        st[rson(p)].data = st[rson(p)].data * st[p].mul_tag % mod;
        st[rson(p)].mul_tag = st[rson(p)].mul_tag * st[p].mul_tag % mod;
        st[rson(p)].add_tag = st[rson(p)].add_tag * st[p].mul_tag % mod;

        st[p].mul_tag = 1;  //乘法标记下推完成，恢复初始值**1**

        st[lson(p)].add_tag +=st[p].add_tag;
        st[lson(p)].data += st[p].add_tag * (st[lson(p)].r - st[lson(p)].l + 1);
        st[rson(p)].add_tag +=st[p].add_tag;
        st[rson(p)].data += st[p].add_tag * (st[rson(p)].r - st[rson(p)].l + 1);

        st[p].add_tag = 0;  //加法标记下推完成，恢复初始值**0**
    }
}

```

需要注意的是，这里的线段树支持了加法和乘法操作。那么在处理加法和乘法操作时，应该注意加法和乘法的先后顺序。先处理乘法，并且用乘法标记更新加法标记（因为在做乘法操作时，加法操作也会被乘以相同的系数），然后再处理加法标记。这样才不会丢失精度以及保持数据记录的准确性。

### 查询操作

要查询指定区间（区间长度>=1）的总和/最大值/最小值等时，可以想到：通过不断向下查询节点，将查询区间[l, r]的问题转化成有限个小区间以及有限个单个元素的并集，即分块思想。

具体实现代码为：

```cpp
long long query(int p, int l, int r)  //p节点序号，l, r范围
{
    //若当前节点包括的数据被完全包含与所求区间，那么直接上推本节点数据
    if (st[p].l >= l && st[p].r <= r) return st[p].data % mod;

    //若没有完全包含，那么就需要用到下方节点，那么下推标记
    lazy_down(p);

    //二分向下查询
    ll ans = 0;
    int mid = (st[p].l + st[p].r) >> 1;
    if (mid >= l) ans += query(lson(p), l, r);
    if (mid <  r) ans += query(rson(p), l, r);

    return ans;
}

```

### 更新操作

加法和乘法更新操作整体步骤基本相同，基本为：如果当前区间被完全包含于修改区间，那么只记录标记；否则继续二分更新。

具体时间代码为：

```cpp
void add(int p, int l, int r, ll k)
{
    //被完全包含
    if (st[p].l >= l && st[p].r <= r)
    {
        st[p].data = (st[p].data + k * (st[p].r - st[p].l + 1)) % mod;
        st[p].add_tag += k;
        return ;
    }
    
	//没有被完全包含
    lazy_down(p);
    int mid = (st[p].l + st[p].r) >> 1;
    if (mid >= l) add(lson(p), l, r, k);
    if (mid <  r) add(rson(p), l, r, k);

    push_up(p);
}
//思路同上函数
void mul(int p, int l, int r, ll k)
{
    if (st[p].l >= l && st[p].r <= r)
    {
        st[p].data = st[p].data * k % mod;
        st[p].mul_tag = st[p].mul_tag * k % mod;
        st[p].add_tag = st[p].add_tag * k % mod;
        return ;
    }

    lazy_down(p);
    int mid = (st[p].l + st[p].r) >> 1;
    if (mid >= l) mul(lson(p), l, r, k);
    if (mid <  r) mul(rson(p), l, r, k);

    push_up(p);
}

```

以上就是常规递归线段树的大体思路和具体实现代码（最值线段树）。总代码被放在了文章最下方。

## 非递归线段树（zkw线段树）

zkw线段树是由神犇zkw在早些时候提出的**非递归式线段树**。在线段树概念的基础上，优化了常规递归线段树的巨大常数，让线段树的常数变得很小，代码量也同时减少了不少。这里只讨论拥有加法标记的非递归线段树。

### 建树操作

使用两个数组，进一步缩小常数（玄学操作）：

```cpp
ll st[MAXN << 2], add[MAXN << 2];  //线段树和标记
int n, N = 1;  //n为原始数据数量，N为**原始数据的起始位置**

inline void build()
{
	for (; N < n + 1; N <<= 1);  //找到原始数据应该在的位置
	for (int i = N + 1; i <= N + n; i++) read(st[i]); //放置原始数据
	for (int i = N - 1; i >= 1; i--) st[i] = st[i << 1] + st[i << 1 | 1];//从下向上构建线段树
}

```

代码量变小了，意味着信息浓度的增加。这短短三个for语句其实蕴含了不少信息量。

在zkw线段树中，我们将原始数据放在线段树数组的最后方+1的位置，这样将有利于之后的各种操作。

二叉树中，对于节点n，n / 2 和 n / 2 + 1分别是n的两个儿子节点。位运算中的表述如上代码，不再赘述。

### 区间分割

在zkw线段树中，我们的查询和更新操作将引入两个特殊的节点（或者说是"节点指针"）s和t。

对于目标区间[l, r]，s一开始指向节点(l - 1)，t一开始指向节点(r + 1)。每个小步骤，s和t都指向他们当前的父亲节点

通过在纸上的模拟，我们可以发现：

从s，t的初始位置，直到s和t在本次步骤中父亲节点相同。这个过程中，如果s是它父亲节点的左节点，那么加上他的右边的兄弟；如果t是它父亲节点的右节点，那么加上它左边的兄弟。那么最后加上的所有节点包含的区间，恰好不重不漏地等价于[l, r]。

在zkw线段树中，我们可以发现：对于一个长度是1（r - l + 1的值）的节点（即叶子节点），每次更新他需要加上1次标记。同理，对于一个长度是n的节点，每次更新他需要加上n个标记。

可以推出，从叶子节点向上，每次需要加上的标记数量将增加一倍——即乘二，或者说右移一位。

### 查询操作

和递归式线段树不同，zkw线段树的每次操作都是直接从底部开始向上操作。并且在操作完成后继续上推到根节点。

```cpp
inline ll query(int l, int r)
{
	int s = N + l - 1, t = N + r + 1;  //s节点和t节点
    //lNum表示当前s包含的标记个数，rNum表示当t包含的标记个数
    //num表示当前层的节点包含的标记个数（从1开始）
	ll lNum = 0, rNum = 0, num = 1, ans = 0;
    
    //如果s和t拥有同一个父亲节点，那么s ^ t == 1，即二进制路径只有最后一位不同
    //那么s ^ t ^ 1 == 0，循环停止
    //每个循环节，s和t上移，num右移一位
	for (; s ^ t ^ 1; s >>= 1, t >>= 1, num <<= 1)
	{
		if (add[s]) ans += add[s] * lNum;  //处理标记
		if (add[t]) ans += add[t] * rNum;
		if (~s & 1) {ans += st[s ^ 1]; lNum += num;}
		if ( t & 1) {ans += st[t ^ 1]; rNum += num;}
        //如果s是父亲的左儿子，二进制最后一位应该是0，取反后&1应该是1
        //同理，t是父亲的右儿子时，t & 1应该是是1
	}
	for (; s; s >>= 1, t >>= 1)  //继续上推到根节点
	{
		ans += add[s] * lNum;
		ans += add[t] * rNum;
	}
	return ans;
}

```

### 更新操作

也是从下向上，一直上推到根节点。每次上推更新自己以应对包含自己的情形。如果s是左||t是右，那么给它的兄弟上标记并更新节点。

```cpp
inline void update(int l, int r, ll k)
{
	int s = N + l - 1, t = N + r + 1; 
	ll lNum = 0, rNum = 0, num = 1;
	for (; s ^ t ^ 1; s >>= 1, t >>= 1, num <<= 1)
	{
		st[s] += k * lNum;  //更新自己
		st[t] += k * rNum;
        //满足条件就更新自己的兄弟
		if (~s & 1) {add[s ^ 1] += k; st[s ^ 1] += k * num; lNum += num;}
		if ( t & 1) {add[t ^ 1] += k; st[t ^ 1] += k * num; rNum += num;}
	}
	for (; s; s >>= 1, t >>= 1)  //上推到根节点
	{
		st[s] += k * lNum;
		st[t] += k * rNum;
	}
}

```

到此，zkw线段树的基本实现已经差不多完成了。

可以看到，zkw线段树比常规线段树的代码量和常数小了很多，但是换来的是每句话信息量的增大。

## 总结

到此，线段树学习小结基本结束。

本小结为自己学习线段树后总结之用，可能并不适合完全不了解线段树的人入门？但是可能会对稍微知道线段树的人有一些帮助。

两个版本的线段树的具体实现代码可以查看[我的Github页](https://github.com/DCTewi/My-Algorithm-Templates/tree/master/5.%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%20-%20Stucture)。

以上。