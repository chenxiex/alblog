---
title: "[蓝桥杯2024国A]最长子段题解"
date: 2025-01-27T16:58:57+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 二分
image: images/cover.webp
---

封面图源见水印。

## 题目描述

给定一个长度为 $n$ 的序列 $(s_1,s_2,\cdots,s_n)$ 和三个数 $a,b,c$，你需要找出一对 $L,R$ 满足如下式子：

$$
\sum\limits_{i=L}^Rs_i>a(bR-cL),1 \le L \le R \le n
$$

即，序列中的第 $L$ 至 $R$ 项之和大于 $a\cdot (b\cdot R - c \cdot L)$，求出满足条件的 $L,R$ 中 $R - L + 1$ 的最大值。

测试数据保证存在这样的一对 $L$ 和 $R$。

### 输入格式

输入的第一行包含四个整数 $n,a,b,c$，相邻整数之间使用一个空格分隔。

第二行包含 $n$ 个整数 $s_1,s_2,\cdots,s_n$，相邻整数之间使用一个空格分隔。

### 输出格式

输出一行包含一个整数表示答案。

### 样例 #1

#### 样例输入 #1

```
4 1 5 6
1 2 3 4
```

#### 样例输出 #1

```
3
```

### 提示

对于 $60\%$ 的评测用例，$n\le 5000$；  
对于所有评测用例，$1\le n\le 3 \times 10^5$，$1\le a,b,c\le 1000$，$|s_i| \le 10^9$。

## 解题思路

这道题第一眼看到算法标签有二分，马上联想到二分区间长度，然后枚举区间位置来检查答案。于是光速写下如下代码：

```cpp
#include <algorithm>
#include <array>
#include <iostream>
using namespace std;
using ll = long long;
const int MAXN = 3e5;
int n;
ll a, b, c;
array<ll, MAXN + 5> s;
array<ll, MAXN + 5> sum;

bool check(int mid)
{
    for (auto i = 1; i + mid - 1 <= n; i++)
    {
        ll l = i, r = i + mid - 1;
        if (sum.at(r) - sum.at(l - 1) > a * (b * r - c * l))
            return true;
    }
    return false;
}

int main()
{
    cin >> n >> a >> b >> c;
    for (auto i = 1; i <= n; i++)
    {
        cin >> s.at(i);
        sum.at(i) = s.at(i) + sum.at(i - 1);
    }
    auto l = 1, r = n, ans = 0;
    while (l <= r)
    {
        auto mid = (l + r) / 2;
        if (check(mid))
        {
            ans = mid;
            l = mid + 1;
        }
        else
        {
            r = mid - 1;
        }
    }
    cout << ans;
    return 0;
}
```

喜提 WA。一开始我还以为是数据范围之类的问题，检查半天，然后猛然发现并非如此。这里要注意二分的一个重要条件，即保证答案是单调的。在上面的代码中，如果一个区间长度被判定为不符合，那么程序就不会搜索比它更长的区间了。但这是不对的。在本题中，一个短区间不符合条件，并不代表长区间也不符合条件。因此，如果希望使用二分搜索，就必须使搜索的目标满足单调条件。那么有没有这样的目标呢？有的有的，这样的目标还有两个，那就是区间的左端点或右端点。但当然不能直接搜索，需要先做一些处理。

我们思考题目给出的条件：$\sum\limits_{i=L}^Rs_i>a(bR-cL),1 \le L \le R \le n$。首先很容易想到用前缀和处理左边的 $\sum\limits_{i=L}^Rs_i$。于是等式左边可以分解为 $sum[r]-sum[l-1]$。这个式子可以移项，将带有 $r$ 的部分和带有 $l$ 的部分分置不等式两边，即 $sum[r]-abr>sum[l-1]-acl$。这暗示我们可以将两个端点分开处理。我们不妨先假设 $r$ 是固定的，思考 $l$ 的情况。根据不等式，$sum[l-1]-acl$ 越小越好，同时根据题目要求区间长度尽量大，那么在 $r$ 固定的情况下，$l$ 显然也是越小越好。那么我们可以得出结论：对于 $l_1<l_2$, 若 $sum[l_1-1]-acl_1<sum[l_2-1]-acl_2$，那么 $l_2$ 是对答案没有贡献的。我们可以遍历一遍数组，计算出 $f[l]=sum[l-1]-acl$ 序列，然后从序列中删除所有对答案没有贡献的元素，最后就会得到一个单调递减的序列 $f$。那么对于这个单调序列，我们就可以用二分搜索来搜索 $l$ 的值。

在具体实现中，我们可以枚举 $r$，那么在单次枚举中 $r$ 就是固定的。在枚举过程中顺便统计 $f[l]=sum[l]-acl$，对于无效的元素，可以将其设置为其前一个有效元素相同的值，这样搜索时就会自动搜索到第一个有效元素了。

## 代码

附上 AC 代码：

```cpp
#include <algorithm>
#include <array>
#include <iostream>
using namespace std;
using ll = long long;
const int MAXN = 3e5;
int n;
ll a, b, c;
array<ll, MAXN + 5> s;
array<ll, MAXN + 5> sum;
array<ll, MAXN + 5> f;

int main()
{
    cin >> n >> a >> b >> c;
    for (auto i = 1; i <= n; i++)
    {
        cin >> s.at(i);
        sum.at(i) = s.at(i) + sum.at(i - 1);
    }
    f.at(0)=1e18;
    auto ans=0;
    for (auto i=1; i<=n; i++)
    {
        f.at(i)=min(f.at(i-1),sum.at(i-1)-a*c*i);
        auto l = 1, r = i;
        while (l <= r)
        {
            auto mid = (l + r) / 2;
            if (sum.at(i)-a*b*i>f.at(mid))
            {
                ans=max(ans,i-mid+1);
                r=mid-1;
            }
            else
            {
                l=mid+1;
            }
        }
    }
    cout << ans;
    return 0;
}
```