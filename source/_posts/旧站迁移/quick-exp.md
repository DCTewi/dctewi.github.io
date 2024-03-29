---
title: '【算法】从群论到快速幂和快速乘'
date: 2018-08-16 23:55:52
tags: [算法,快速幂,快速乘,旧文归档]
cover: https://s-sh-2563-tewi-box.oss.dogecdn.com/img/blog-header/minoto-paint.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
换一个角度看快速幂和快速乘。

<!-- more -->

**群**在抽象代数中具有基本的重要地位。许多代数结构，包括环、域和向量空间都可以看作是在群的基础上添加新的运算和公理而形成的。群的概念在数学的很多分支都有出现，而且群论的研究方法也对抽象代数的其他分支有重要影响。

在数学中，群是由一个**集合**以及一个**二元运算**所组成的，符合以下四个性质（称为“**群公理**”）的代数结构。即**封闭性**，**结合律**，**单位元**和**逆元存在**。

对于**群$(G,·)$**，该群由一个集合G和一个二元运算·组成。对于该群，群公理表现为：

  1. 封闭性： 对于任意a, b ∈ G，有a · b ∈ G 。
  2. 结合律： 对于任意a, b, c ∈ G，有(a · b) · c = a · (b · c) 。
  3. 单位元： 存在e ∈ G，使得对于所有元素a ∈ G总有 e · a = a · e = a成立。
  4. 逆元存在： 对于任意a ∈ G，存在元素b使得总有a · b = b · a = e，此处e为单位元。

**幺半群**是一种类似群的结构。幺半群不一定满足逆元存在公理，但是其余的三个群公理均满足。即幺半群$(G, *)$满足**{封闭性， 结合律， 单位元}**。又因为幺半群的要求比群要少，所以所有的群都是幺半群。幺半群的例子:(非负整数，加法运算)，(正整数，乘法运算)。

**<del>所以，这和标题里的快速幂和快速乘又有什么关系呢？</del>**

<del>因为，</del>快速幂算法就可以从幺半群的角度来分析。

对于幺半群$<N^*, *>$(N\*为非零自然数，\*是乘法运算)，计算a的k次幂，当k = 0时为1，其余情况为k个n相乘。根据结合律（强行），我们可以在至多$O(log(k))$的时间内把k拆成二进制下的1，然后使用至多$O(log(k))$次乘法合并这些幂次。

用代码表达会更直观一些：

```cpp
typedef long long ll;
ll q_exp(ll a, ll k, ll mod)
{
	if (!b) return 1; //任何实数的0次幂都是1;

	ll ans = 1; //结果变量
	while (k) //拆k
	{
		if (k & 1) ans = ans * a % mod; //如果现在的k是个奇数，则ans多乘一个a;
		a = a * a % mod; //a自乘
		k >>= 1; //继续拆k
	}
	
	return ans; //返回结果
}
```

在这个过程中，我们用到了一下几个性质：

  1. 乘法 满足封闭性
  2. 乘法 满足结合律
  3. 乘法 下有单位元(即1)

即我们在不知不觉中已经使用到了幺半群的相关内容。而如果我们稍加思考，将上述幺半群$<N^*, *>$魔改成$<Z, +>$呢？因为加法也满足以上的几个性质(单位元是0)，所以我们就引申到了另一个神奇的算法——快速乘。这种神奇的算法在罗宾米勒算法中，为了防止$a * b % mod$时溢出而使用过。当$a * b$的结果连`long long`都放不下的时候，求$a * b % mod$ 的结果就需要用到这种方法。

```cpp
typedef long long ll;
ll q_mul(ll a, ll b, ll mod)
{
	if (!b) return 0; //任何数乘0都是0

	ll ans = 0; //结果变量
	while (b) //拆b
	{
		if (b & 1) ans = (ans + a) % mod; //如果现在的b是个奇数，则ans多加一个a;
		a = (a + a) % mod; //a自加
		b >>= 1; //继续拆b
	}
	
	return ans;//返回结果
}
```

这样我们就得到了快速乘算法的代码。

以上，快速幂和快速乘其实就这么结束了，所以抽象代数真的很厉害。