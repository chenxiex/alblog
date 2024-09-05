---
title: "幂的计算方法"
date: 2024-08-29T20:18:56+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 快速幂
    - 数学
image: images/cover.webp
---
封面图：[https://www.bilibili.com/opus/967606108335112258](https://www.bilibili.com/opus/967606108335112258)

## 前言
本篇介绍一些算法竞赛中常见的幂的计算方法。本篇并不重点介绍快速幂等基本算法，而是针对一些特殊情况，介绍一些可用于优化的数学定理，以在快速幂的时间复杂度都无法满足题目要求的情况下，进一步优化算法。

## 前置知识
### 同余的运算性质
设 $m$ 是一个正整数，$a_1, a_2, b_1, b_2$ 是 $4$ 个整数。如果
$$
a_1 \equiv b_1 \pmod{m}, \quad a_2 \equiv b_2 \pmod{m}
$$
则
1. $a_1 + a_2 \equiv b_1 + b_2 \pmod{m}$
2. $a_1 \cdot a_2 \equiv b_1 \cdot b_2 \pmod{m}$

需要注意的是，同余的运算性质中没有针对除法的性质。

### 快速幂
C++ 本身有提供 `pow()` 函数，但该函数是浮点数的幂运算，且不支持取模运算。在竞赛中，我们常常遇到大整数的幂运算和模幂运算，这需要自己实现快速幂算法。

Python 的 `pow()` 函数支持大整数的幂运算和模幂运算，且在我的测试中，速度比自己实现的快速幂要快。因此，如果是 Python 选手，建议直接使用 `pow()` 函数。

常见的快速幂算法，也就是模重复平方算法。快速模幂运算的解释，网络上有很多，不是本文的重点。简单来说，快速幂就是将指数表示成二进制（$2^n+2^{n-1}+...+2^1+2^0$）的形式，然后通过不断平方基数来减少乘法的次数。以下是一个简单的快速幂的实现：

```cpp
long long quick_pow(long long a, long long b, long long mod) {
    long long ans = 1;
    while (b) {
        if (b & 1) {
            ans = ans * a % mod;
        }
        a = a * a % mod;
        b >>= 1;
    }
    return ans;
}
```
快速计算模幂运算的算法不只这一种。针对这一种算法，也有一些常数级别的优化方法。如果感兴趣，也可以自行了解其他算法。

### 欧拉函数
欧拉函数 $\varphi(n)$ 表示小于等于 $n$ 的正整数中与 $n$ 互质的数的个数。
#### 欧拉函数的性质
1. 如果 $p$ 是质数，则 $\varphi(p) = p - 1$
2. 如果 $p$ 是质数，则 $\varphi(p^k) = p^k - p^{k-1}$
3. 设 $m,n$ 是互质的两个正整数，则 $\varphi(mn) = \varphi(m) \cdot \varphi(n)$。特别地，如果 $m,n$ 是质数，则 $\varphi(mn) = (m-1)(n-1)$
4. 设正整数 $m$ 的标准因数分解式为 $m = \underset{p|m}{\prod} p^k = p_1^{k_1} \cdot p_2^{k_2} \cdot \cdots \cdot p_n^{k_n}$，则 $\varphi(m) = \underset{p|m}{\prod}\varphi(p^k) = m\underset{p|m}{\prod}(1-\frac{1}{p}) = m \cdot (1 - \frac{1}{p_1}) \cdot (1 - \frac{1}{p_2}) \cdot \cdots \cdot (1 - \frac{1}{p_n})$
5. 设 $m$ 是一个正整数，则 $\underset{d|m}{\sum} \varphi(d) = m$。

以下给出求解欧拉函数的 C++ 代码，该代码利用了性质4和性质2：
```cpp
long long phi(long long a)
{
    long long result = a;
    for (int i = 2; i * i <= a; i++)
    {
        if (a % i == 0)
        {
            while (a % i == 0)
                a /= i;
            result -= result / i;
        }
    }
    if (a > 1)
        result -= result / a;
    return result;
}
```

## 幂的计算方法
### 欧拉定理
设 $m$ 是大于 $1$ 的正整数，$a$ 是与 $m$ 互质的整数，则
$$
a^{\varphi(m)} \equiv 1 \pmod{m}
$$
该定理提示了，如果题目中存在条件基数 $a$ 和模数 $m$ 互质，那么指数 $k$ 就可以简化为 $k \mod{\varphi(m)}$。

### 费马小定理
设 $p$ 是一个质数，则对任意整数 $a$，有
$$
a^p \equiv a \pmod{p}
$$
推论：设 $p$ 是一个质数，则对任意整数 $a$，以及对任意正整数 $t,k$，有
$$
a^{k \cdot (p-1) + t} \equiv a^t \pmod{p}
$$
该推论提示了，如果题目中存在条件模数 $p$ 为质数，那么指数 $k$ 就可以简化为 $k \mod{(p-1)}$。容易发现，费马小定理是欧拉定理在模数为质数时的特殊情况。

### 拓展欧拉定理
欧拉定理要求基数 $a$ 与模数 $m$ 互质，而拓展欧拉定理则放宽了这一条件。设 $m$ 是大于 $1$ 的正整数，$a$ 是任意整数，$k \geq \varphi(m)$，则
$$
a^k \equiv a^{k \mod{\varphi(m)} + \varphi(m)} \pmod{m}
$$
如果 $k<\varphi(m)$，那么只需要直接计算 $a^k$ 即可。该定理去除了欧拉定理中 $a$ 与 $m$ 互质的要求。因此，通过扩展欧拉定理，我们就能将几乎所有情况下的指数简化到小于 $2\varphi(m)$。

## 实操演练
[洛谷P10414](https://www.luogu.com.cn/problem/P10414)

### [蓝桥杯 2023 国 A] 2023 次方

#### 题目描述

求 $2^{(3^{(4^{(\ldots ^{2023}})})}$ 的值对 $2023$ 取模的结果。

注: 上式都是指数，可写为 $2** (3**(4**(\ldots 2023\ldots))$ 其中 $**$ 表示指数。

这是一道结果填空的题，你只需要算出结果后提交即可。本题的结果为一个整数，在提交答案时只填写这个整数，填写多余的内容将无法得分。

#### 输入格式

#### 输出格式

### 解题思路
注意到，$2023=7 \times  17 \times 17$，并非质数；通过计算可知，$\varphi(2023)=1632$，而 $gcd(3,1632)\neq 1$。因此，不能使用费马小定理和欧拉定理。考虑使用扩展欧拉定理求解。本题目中要求解的算式也比较复杂，建议动手算算前几项来确定递归关系。

### 代码实现
```cpp
#include <iostream>
using namespace std;
const int n = 2023, mod = 2023;
int phim;
long long phi(long long a)
{
    long long result = a;
    for (int i = 2; i * i <= a; i++)
    {
        if (a % i == 0)
        {
            while (a % i == 0)
                a /= i;
            result -= result / i;
        }
    }
    if (a > 1)
        result -= result / a;
    return result;
}
long long fpow(long long a, long long b, long long mod)
{
    long long ans = 1;
    while (b > 0)
    {
        if (b & 1)
            ans = ans * a % mod;
        a = a * a % mod;
        b >>= 1;
    }
    return ans;
}
long long f(long long a, long long m)
{
    if (a == n)
        return a;
    long long phim = phi(m);
    return fpow(a, f(a + 1, phim) + phim, m);
}
int main()
{
    cout << f(2, mod);
}
```