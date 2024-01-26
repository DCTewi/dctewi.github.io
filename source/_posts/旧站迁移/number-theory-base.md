---
title: '【算法】数论基础小结'
date: 2019-02-19 11:48:31
tags: [数论,算法,最大公因数,素数,组合数]
cover: https://s2.ax1x.com/2019/02/19/kgpPIO.jpg
coverWidth: 1280
coverHeight: 726
categories:
- 旧站迁移
---
> 数论是纯粹数学的分支之一，主要研究整数的性质。被誉为“最纯”的数学领域。

<!-- more -->

总结了一些在寒假集训的时候学到的数论知识，不按难度排序。

## 关于素数

素数，指除了1和它本身没有其他因数的正整数。

### 朴素素数判定

即从定义出发，遍历从2到根号n所有数。如果存在一个数字是他的因数，则这个数不是素数。

实现代码如下：

```cpp
bool checkPrime(int n)
{
    for (int i = 2; i < sqrt(n); i++)
    {
        if (n % i == 0) return false;
    }
    return true;
}

```

这种方法太过朴实，一般情况下不会用到。时间复杂度```O(√n)```。

### 埃氏素数筛法

一种较为高效的判断素数的方法。能够一次性地筛选出某个区间的素数。

其中心思想是：如果某个数x是a的倍数，那么x一定不是素数。每次得到一个素数，则把他在这个区间内的所有素数都标记为合数。

实现代码如下：

```cpp
const int MAXN = 1e4 + 5;
bool notPrime[MAXN];
void checkPrimeTo(int uplimit)
{
    notPrime[0] = notPrime[1] = 1;
    for (int i = 2; i <= uplimit; i++)
    {
        if (!notPrime[i])
        {
            for (int j = 2 * i; j <= uplimit; j += i)
            {
                notPrime[j] = 1;
            }
        }
    }
}

```

```notPrime[n] == 1```则表示n不是素数。这个算法看似是```O(n ^ 2)```的复杂度，其实随着i的不断增大，j的循环范围在不断变小。在整个测试的范围内，可以**近似于**线性复杂度。实际时间复杂度为```O(nloglog(n))```。

### 线性欧拉筛法

在埃氏筛法的基础上，通过巧妙地限制每次筛去的数字数量，保证了**每个合数只会被它的最小质因数筛去**，因此每个数只会被标记一次。

实现代码如下：

```cpp
const int MAXN = 1e7 + 5;
int notPrime[MAXN], primes[MAXN], top = 0;
void checkPrimeTo(int uplimit)
{
    notPrime[0] = notPrime[1] = 1;
    for (int i = 2; i <= uplimit; i++)
    {
        if (!notPrime[i]) primes[top++] = i;
        for (int j = 0; j < top; j++)
        {
            if (i * primes[j] > uplimit) break;
            check[i * primes[j]] = 1;
            if (i % primes[j] == 0) break;
        }
    }
}

```

```primes[]```记录的是筛出的素数们，通过遍记录素数，来保证每个数只会被便利一次。时间复杂度```O(n)```。

### Miller-Rabin素性测试

首先需要了解**费马小定理**：

```yaml
对于素数 p 和任意整数 a ，若gcd(a, p) == 1，
则有 a ^ p ≡ a (mod p)，
即 a ^ (p - 1) ≡ 1 (mod p)。
```

那么，是否可以反过来利用这个定理来判断素数呢？基本可以。

但是有一类合数，用任何小于他们的质数为底进行判定，都满足这个定理。这种合数叫做伪素数，最小的伪素数是341。为了排除伪素数，我们引入了**二次探测定理**：

```yaml
对于素数 p ，则 x ^ 2 ≡ 1 (mod p) => ( x == 1 || x == p - 1 )
```

则对于```a ^ (x - 1) = 1 (mod p)```成立，若```x - 1```是奇数，则不再继续判断。否则继续判断是否```a ^ ((x - 1) / 2) = 1 或 -1 (mod p)```成立。如果不等于1或者-1则返回```false```，如果等于-1则返回`true`，如果等于1则继续向下判断。

选择前7个素数作为a，在`[2, 1e18]`内也就只有两三个可以通过Miller-Rabin测试的强伪素数了。

实现代码：（`q_exp()`为快速幂，`q_mul()`为快速乘）

```cpp
typedef unsigned long long ll;
const int TIMES = 20; // 测试轮数

ll q_exp(ll a, ll k, ll mod)
{
	if (!k) return 1;
	ll ans = 1;
	while (k)
	{
		if (k & 1) ans = ans * a % mod;
		a = a * a % mod; k >>= 1;
	}
	return ans;
}
ll q_mul(ll a, ll b, ll mod)
{
	if (!b) return 0;
	ll ans = 0;
	while (b)
	{
		if (b & 1) ans = (ans + a) % mod;
		a = (a + a) % mod; b >>= 1;
	}
	return ans;
}

inline ll random(ll n)
{
    return ((double)rand() / RAND_MAX * n + 0.5); // RAND_MAX 定义于头文件
}

bool witness(ll a, ll n) // 二次探测
{
    ll tem = n - 1;
    int j = 0;
    while (!(tem & 1))
    {
        tem /= 2; j++;
    }

    ll x = q_exp(a, tem, n);
    if (x == 1 || x == n - 1) return true;
    
    while (j--)
    {
        x = q_mul(x, x, n);
        if (x == n - 1) return true;
    }
    
    return false;
}

bool miller_rabin(ll p) // 随机生成a来二次探测
{
    if (p == 2) return true;
    if (p < 2 || !(p & 1)) return true; // 剪枝
    
    for (int i = 1; i < TIMES; i++)
    {
        ll a = random(p - 1) + 1;
        if (!witness(a, p))
        {
            return false;
        }
    }
    return true;
}

int main()
{
    ll n; cin>>n;
    miller_rabin(n)? puts("It's a prime."): puts("It's not a prime");
}

```

Miller-Rabin的时间主要花在了计算`a ^ (p - 1) mod p`上，总体时间复杂度是`O(logn)`。

## 关于最大公因数

对于最大的整数n，使得n同时是a和b的因子，则称n是a和b的最大公因数(Greatest common divisor)。

性质：`gcd(a, b) * lcm(a, b) = a * b`。

### 递归求gcd

辗转相除法，直接上代码：

```cpp
typedef long long ll;
inline ll gcd(int a, int b)
{
    return b? gcd(b, a % b): a;
}

```

### 扩展欧几里德算法(exgcd)

扩展欧几里德算法是用来解决`已知a, b求解一组x, y,使它们满足: ax + by = gcd(a, b) = d`的算法。

代码（接上部分代码）：

```cpp
ll exgcd(ll a, ll b, ll &x, ll &y)
{
    if (!b)
    {
        x = 1; y = 0; return a;
    }
    ll r = exgcd(b, a % b, x, y);
    ll tem = y;
    y = x - a / b * y;
    x = tem;
    return r;
}
```

## 关于组合数

[组合数的定义](https://baike.baidu.com/item/%E7%BB%84%E5%90%88%E6%95%B0)

### 杨辉三角求组合数

根据推论：

```
C(m, n) = C(m, n - 1) + C(m - 1, n - 1)
```

可以预处理打表一个杨辉三角来求组合数。

代码大致为：

```cpp
typedef long long ll;
const int MAXN = 1e4 + 5;
ll c[MAXN][MAXN];
void getC(int uplimit)
{
    c[0][0] = c[1][0] = c[1][1] = 1;
    for (int i = 2; i <= uplimit; i++)
    {
        c[i][0] = 1;
        for (int j = 1; j <= i; i++)
        {
            c[i][j] = c[i][j - 1] + c[i - 1][j - 1];
        }
    }
}

```

没啥注意的要点，时间复杂度`O(n ^ 2)`。

### 逆元打表求组合数

步骤大概如下：

1. 打表出`1! ~ n!` - `O(n)`。
2. 求每个阶乘的逆元 - `O(nlogn)`。
3. 直接得：`C(m, n) = n! / (m! * (n - m)!)` - `O(1)`。

总体复杂度是`O(nlogn)`。

### Lucas定理

> 在数论中，Lucas定理用于计算二项式系数(m, n)被质数 p 除的所得的余数。

定理内容：对于非负整数 m 和 n 以及素数 p ，同余式：

```
(m, n) ≡ Π(i = 0 -> k) (m[i], n[i]) (mod p)
```

其中，`m[i]`，`n[i]`是 `m` 和 `n` 的 p 进制展开，当`m < n` 时，二项式系数`(m, n) = 0`。

以后我学会Latex之后可能会重新写一下这里……这样平摊有点抽象_(:3。

这个方法使用递归，用于n和m巨大，但是p不太大且p为质数的时候。可直接求得：

```
C(m, n) ≡ C(m % p, n % p) * C(m / p, n / p)
```

代码就不写了。

### 分块打表求组合数

玄学操作，大概知道有这么一种方法就可以了。时间复杂度`O(玄学)`。

## 零散的知识点

### 唯一分解定理

任何一个大于1的自然数 N,如果N不为质数，那么N可以唯一分解成有限个质数的乘积`N = P1 ^ a1 * P2 ^ a2 * P3 ^ a3 ... Pn ^ an`，这里`P1 < P2 < P3 <...< Pn`且均为质数，其中指数`ai`是正整数。这样的分解称为 N 的标准分解式。

### 同余定理

#### 加减乘同余

模意义下，加减乘都不变，即：

```
(a % c + b % c) % c == (a + b) % c
```

减法、乘法同理。

#### 逆元同余

逆元：

```
任意 x ∈ (1, p - 1)，存在 y 使 x * y % p == 1，称 y 是 x 在 mod p 意义下的逆元
```

则：

```
(a % c) / (x % c) != (a / x) % c 
```

但是

```
(a % c) * (y % c) % c == (a / x) % c
```

### 欧拉函数/欧拉降幂

略（逃）



以上就是寒假集训中学到的一些数论知识点。