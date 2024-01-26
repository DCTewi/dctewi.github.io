---
title: '【算法】没啥用的判圈算法两则'
date: 2020-03-15 23:37:04
tags: [算法,笔记,图论]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/kaf-city.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---

如题，这里记录了两则没啥用的判圈算法。

<!-- more -->

对于“判断有向图中是否存在环”这类问题，竞赛中常见的解决办法一般都是带标记深搜或者拓扑排序后删点，在邻接表或者前向星下这两种方法的时间复杂度都是线性，所以一般够用。

但最近在某个沙雕视频里了解到了一个奥妙算法，叫做 Floyd Cycle Detection，可以在线性时间，常数空间的条件下判断环是否存在。但是竞赛题没办法这么卡空间，了解下这算法也就图一乐。

## Floyd Cycle Detection

> Floyd 判圈算法 (Floyd Cycle Detection Algorithm)，又称龟兔赛跑算法 (Tortoise and Hare Algorithm)，是一个可以在有限状态机、迭代函数或者链表上判断是否存在环，求出该环的起点与长度的算法。
>
> 如果有限状态机、迭代函数或者链表上存在环，那么在某个环上以不同速度前进的 2 个指针必定会在某个时刻相遇。同时显然地，如果从同一个起点(即使这个起点不在某个环上)同时开始以不同速度前进的 2 个指针最终相遇，那么可以判定存在一个环，且可以求出 2 者相遇处所在的环的起点与长度。

这是 Wikipedia 上的算法描述。简而言之，如果图有环，那么从图上某一点同时出发的速度不同的两个指针，一定会在快的指针比慢的指针多跑整数倍长度的某一时刻相遇。相反，如果快的点遍历完了图，那么则说明图上没有环。

Floyd 判圈的具体做法是令快指针每次走两步，慢指针每次走一步。两个指针相遇之后定住一个让另一个跑一圈就可以求出环长度了。

从慢指针到环上开始，慢指针最多只会走一圈两个指针就会相遇。所以时间复杂度是线性$O(m+n)$，即两个指针的步数和。只创建了两个指针和环长，所以空间复杂度是$O(1)$。

代码大概就这样（临时手搓，没有验证）：

```c++
int floyd_cycle_detection(Node *head)
{
    Node *slow = head, *quik = head;
    int len = 1;
    while (1)
    {
        if (quik == nullptr) return -1;
        if (slow == quik) break;
        
        slow = slow->next;
        quik = quik->next->next;
    }
    while (slow->next != quik) slow = slow->next, len++;
    return len;
}
```

## Brent's Cycle Detection

Brent's Cycle Detection 算法是上边那个的优化，减少了常数，平均消耗时间减少了36%（据说）。

他和 Floyd 判圈的不同在于快指针每次的移动长度变成了$2^i$，慢指针不移动，而是若每次快节点移动的过程中两者没有相遇，则慢指针直接移动到快节点的位置。这样如果有环，快指针在最后一次移动过程中的步数就是环长度。若无环，则慢指针一直在“追逐”快指针，无法相遇。

也是挺好理解的，代码大概这样：

```c++
int brent_cycle_detection(Node *head)
{
    Node *slow = head, *quik = head;
    int len = 0, now = 1;
    while (1)
    {
        len++; quik = quik->next;
        if (quik == nullptr) return -1;
        if (slow == quik) return len;
        
        if (len >= now)
        {
            len = 0; now <<= 1;
            slow = quik;
        }
    }
}
```

## 最后

虽然这俩算法可能没啥实战作用，但是某种意义上也能感受到一个学者应该拥有的那种对某个问题吹毛求疵的探索态度，值得我们学习。

~~但是这俩算法确实没啥用。~~

**题外话：** 彩虹社新专「SMASH The PAINT!!」太顶了，Sasaki的那首[「煽動海獣ダイパンダ」](https://www.bilibili.com/video/av93933353)过于清楚，ガチ了。

