---
title: "[蓝桥杯2024省A]成绩统计题解"
date: 2025-01-21T15:10:48+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 二分
image: images/cover.webp
---

封面图源见水印

## 题目描述
[洛谷P10389](https://www.luogu.com.cn/problem/P10389)

小蓝的班上有 $n$ 个人，一次考试之后小蓝想统计同学们的成绩，第 $i$ 名同学的成绩为 $a_i$。当小蓝统计完前 $x$ 名同学的成绩后，他可以从 $1 \sim x$ 中选出任意 $k$ 名同学的成绩，计算出这 $k$ 个成绩的方差。小蓝至少要检查多少个人的成
绩，才有可能选出 $k$ 名同学，他们的方差小于一个给定的值 $T$？
提示：$k$ 个数 $v_1, v_2, \cdots , v_k$ 的方差 $\sigma^2$ 定义为：$\sigma^2=\dfrac  {\sum_{i=1}^k(v_i-\bar v)^2} k$，其中 $\bar v$ 表示
$v_i$ 的平均值，$\bar v = \dfrac {\sum_{i=1}^k v_i} k$。

### 输入格式

输入的第一行包含三个正整数 $n, k, T $，相邻整数之间使用一个空格分隔。

第二行包含 $n$ 个正整数 $a_1, a2, \cdots, a_n$ ，相邻整数之间使用一个空格分隔。

### 输出格式

输出一行包含一个整数表示答案。如果不能满足条件，输出 $-1$ 。

### 样例 #1

#### 样例输入 #1

```
5 3 1
3 2 5 2 3
```

#### 样例输出 #1

```
4
```

### 提示

检查完前三名同学的成绩后，只能选出 $3, 2, 5 $，方差为 $1.56 $；

检查完前四名同学的成绩后，可以选出 $3, 2, 2 $，方差为 $0.22 < 1 $，所以答案为 $4 $。

对于 $10\%$ 的评测用例，保证 $1 ≤ n, k ≤ 10^2$；  
对于 $30\%$ 的评测用例，保证 $1 ≤ n, k ≤ 10^3$ ；  
对于所有评测用例，保证 $1 ≤ n, k ≤ 10^5 $，$1 ≤ T ≤ 2
^{31} -1 $，$1 ≤ a_i ≤ n $。

## 解题思路

在蓝桥杯做的第二道二分答案的题，然后我又没做出来 QAQ。

这道题不必过分拘泥于方差如何如何。首先要想到最终答案是要求得一个尽量小的 $x$，$x$ 的二分搜索思路非常简单：二分搜索 $x$，判断是否符合要求，如果符合要求就在左区间继续搜索，否则在右区间搜索。

在确定了这个二分搜索的大方向之后，再考虑如何高效实现判断。要求 $x$ 个数中任意 $k$ 个数的方差，乍看起来复杂度很高，会超时。我最开始也是被这个复杂度吓退了，导致没往这个方向想。但实际上这个复杂度是完全可以接受的。首先在 $x$ 尽量小的基础上，要求方差尽量低，这样才可能小于 $T$。因此容易想到对前 $x$ 个数进行排序，排序之后相邻的数的方差就会比较小。排序的时间复杂度是 $O(nlogn)$。然后就需要挪动一个大小为 $k$ 的窗口，求窗口内的方差。挪动窗口本身是一个接近 $O(n)$ 的复杂度，因此求方差的复杂度不能太高。考虑到我们是在一个挪动的窗口内求方差，尝试通过前缀和进行优化。使用完全平方公式展开求方差的式子，得到 $\sigma^2=\frac{\sum_{i=1}^{k}{v_i^2}-2\sum_{i=1}^{k}{v_i}\bar{v}+k\bar{v}^2}{k}$。其中 $\sum_{i=1}^k{v_i^2}$，$\sum_{i=1}^k{v_i}$ 和 $\bar{v}$ 实际上都可以通过前缀和得到，那么求方差的时间复杂度就是 $O(1)$。那么整个判断的时间复杂度就是排序 $O(nlogn)$ + 求前缀和 $O(n)$ + 挪动窗口求方差 $O(n)$，最终大约是 $O(nlogn)$ 的复杂度。而二分搜索本身是 $O(logn)$ 的复杂度，所以整个算法的复杂度就是 $O(nlog^2n)$。完全在题目要求范围内。

## 代码

以下附上代码。该代码在蓝桥杯官网通过，但在洛谷无法通过，可能是精度问题。

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>
using namespace std;

using ll = long long;
using ld = long double;

int main()
{
    int n, k;
    ld t;
    cin >> n >> k >> t;
    vector<ll> a;
    for (auto i = 0; i < n; i++)
    {
        ll temp;
        cin >> temp;
        a.push_back(temp);
    }
    auto ans = distance(a.begin(), a.end()) + 1;
    auto l = a.begin(), r = a.end();
    while (l < r && distance(a.begin(), r) > k)
    {
        auto m = l + (r - l) / 2;
        vector<ll> b(a.begin(), m + 1);
        sort(b.begin(), b.end());
        vector<ll> sum, powsum;
        bool flag = false;
        for (auto i : b)
        {
            sum.push_back(sum.empty() ? i : i + *sum.crbegin());
            powsum.push_back(powsum.empty() ? i * i : i * i + *powsum.crbegin());
        }
        for (auto i = 0; i + k - 1 < b.size(); i++)
        {
            auto j = i + k - 1;
            auto sumk = sum[j] - (i > 0 ? sum[i - 1] : 0);
            auto powsumk = powsum[j] - (i > 0 ? powsum[i - 1] : 0);
            auto avg = ld(sumk) / k;
            auto var = (powsumk - 2 * sumk * avg + k * avg * avg) / k;
            if (var < t)
            {
                flag = true;
                break;
            }
        }
        if (flag)
        {
            ans = min(distance(a.begin(), m), ans);
            r = m;
        }
        else
        {
            l = m + 1;
        }
    }
    if (ans == distance(a.begin(), a.end()) + 1)
        cout << -1 << endl;
    else
        cout << ans + 1 << endl;
    return 0;
}
```