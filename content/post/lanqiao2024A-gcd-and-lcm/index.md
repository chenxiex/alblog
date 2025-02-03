---
title: "[蓝桥杯2024国A]gcd与lcm"
date: 2025-02-02T22:39:12+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 数学
    - 快速幂
image: images/cover.webp
---

封面图：[https://www.pixiv.net/artworks/126731977](https://www.pixiv.net/artworks/126731977)

## 题目描述

给定两个数 $x,y$，求有多少种不同的长度为 $n$ 的序列 $(a_1,a_2,\cdots,a_n)$，其所有元素的最大公约数为 $x$ 且最小公倍数为 $y$。

两个序列 $(a_1,a_2,\cdots,a_n)$ 与 $(b_1,b_2,\cdots,b_n)$ 不同，是指存在至少一个位置 $i$ 满足 $a_i\neq b_i$。

由于答案可能很大，请输出答案对 $998\ 244\ 353$ 取模后的结果。

### 输入格式

输入的第一行包含一个整数 $Q$ 表示询问次数。

接下来 $Q$ 行，每行包含三个整数 $x,y,n$ 表示一组询问，相邻整数之间使用一个空格分隔。对于每个询问，保证至少存在一个满足条件的序列。

### 输出格式

输出 $Q$ 行，每行包含一个整数，依次表示每个询问的答案。

### 输入输出样例 #1

#### 输入 #1

```
3
3 6 2
12 144 3
233 251640 10
```

#### 输出 #1

```
2
72
905954656
```

### 说明/提示

对于 $40\%$ 的评测用例，$n\le 30$；  
对于 $70\%$ 的评测用例，$n\le 5000$；  
对于所有评测用例，$1\le Q\le 100$，$2\le n\le 10^5$，$1\le x,y\le 10^9$。

## 解题思路

在蓝桥杯做的第二道排列组合，非常简单，然而没能做出来 QAQ。

这道题题目里虽然有 gcd 和 lcm，但是其实并不需要求 gcd 或 lcm。由题目条件可以想到，$y=0\mod x$。令 $t=y\div x$，那么 $t=0\mod a_i$。若对 $t$ 分解质因数得到 $t=p_1^{k_1}p_2^{k_2}p_3^{k_3}\cdots$，那么 $a_i$ 实际上就是这些质因数的排列组合。考虑质因数 $p_i$，它在每个 $a$ 中的次数都可能为 $[0,k_i]$，共 $k_i+1$ 种可能。那么考虑所有 $a_i$，根据乘法原理，就有 $(k_i+1)^n$ 种可能。但是，有两种情况是不允许的：所有 $a_i$ 中 $p_i$ 的次数都不为 $0$，这样的话它们的最大公因数就是 $x\cdot p_i^{j_i}$，$j_i$ 为所有 $a_i$ 中 $p_i$ 最低的次数；同理，如果所有 $a_i$ 中 $p_i$ 的次数都不为 $k_i$，则会导致最小公倍数小于 $y$。因此我们需要减去这两种情况。第一种情况时，$p_i$ 的次数可能为 $[1,k_i]$，共 $k_i$ 种可能。第二种情况时，$p_i$ 的次数可能为 $[0,k_i-1]$，也是 $k_i$ 种可能。考虑所有 $a_i$ 就是 $k_i^n$ 种可能。因此我们要减去 $2k_i^n$。这时候注意到次数为 $[1,k_i-1]$ 的情况被减去了两次，因此再加上 $2(k_i-1)^n$。单个 $p_i$ 的次数的所有可能数就为 $(k_i+1)^n-2k_i^n+(k_i-1)^n$。所有 $p_i$ 之间实际上互不影响，因此最终总可能数就是 $\sum_{i=1}^n(k_i+1)^n-2k_i^n+(k_i-1)^n$。

因为需要求模，实现的时候需要手写快速幂。

## 代码

附上 AC 代码。

```cpp
#include<iostream>
#include<utility>
#include<vector>
#include<cmath>
using namespace std;
using ll=long long;
using pf=pair<int,int>;
const auto M=998244353;
const ll MAXX=1e9;
int q;

bool isprime(ll a)
{
    if (a==0||a==1) return false;
    if (a==2) return true;
    for (ll i=2;i*i<=a;i++)
    {
        if (a%i==0) return false;
    }
    return true;
}
vector<pf> fac(int t)
{
    vector<pf> result;
    for (auto i=2;i<=t;i++)
    {
        if (t%i==0&&isprime(i))
        {
            int p=0;
            while (t%i==0)
            {
                t/=i;
                p++;
            }
            result.emplace_back(t,p);
        }
    }
    return result;
}
ll qpow(ll a,ll p)
{
   ll t=1,ans=1,at=a;
   while(t<=p)
   {
       if (p&t) ans=ans*at%M;
       t<<=1;
       at=at*at%M;
   }
   return ans;
}

int main()
{
    cin>>q;
    for (auto i=0; i<q; i++)
    {
        int x,y;
        int n;
        cin>>x>>y>>n;
        auto t=y/x;
        vector<pf> tpf=fac(t);
        ll ans=1;
        for (auto j:tpf)
        {
            ll tmp=qpow(1+j.second,n);
            tmp=(tmp-qpow(j.second,n));
            tmp=(tmp-qpow(j.second,n));
            tmp=(tmp+qpow(j.second-1,n))%M;
            tmp=(tmp+M)%M;
            ans*=tmp;
            ans%=M;
        }
        cout<<ans<<endl;
    }
}
```