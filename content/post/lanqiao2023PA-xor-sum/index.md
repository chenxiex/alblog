---
title: "[蓝桥杯2023省A]异或和之和"
date: 2025-01-18T16:50:42+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 位运算
image: images/cover.webp
---

## 题目描述

[洛谷P9236](https://www.luogu.com.cn/problem/P9236)

给定一个数组 $A_i$，分别求其每个子段的异或和，并求出它们的和。或者说，对于每组满足 $1 \leq L \leq R \leq n$ 的 $L,R$，求出数组中第 $L$ 至第 $R$ 个元素的异或和。然后输出每组 $L,R$ 得到的结果加起来的值。

### 输入格式

输入的第一行包含一个整数 $n$ 。

第二行包含 $n$ 个整数 $A_i$，相邻整数之间使用一个空格分隔。

### 输出格式

输出一行包含一个整数表示答案。

### 样例 #1

#### 样例输入 #1

```
5
1 2 3 4 5
```

#### 样例输出 #1

```
39
```

### 提示

##### 【评测用例规模与约定】

对于 $30 \%$ 的评测用例，$n \leq 300$；

对于 $60 \%$ 的评测用例，$n \leq 5000$;

对于所有评测用例，$1 \leq n \leq 10^5$，$0 \leq A_i \leq 2^{20}$。

## 解题思路

首先想到前缀和，然后枚举 $L$ 和 $R$ ，时间复杂度 $O(n^2)$ ，代码非常简单，可以拿下60分。代码如下：

```cpp
#include <iostream>
using namespace std;
const int MAXN=100000;
int n,a[MAXN+5];
int sum[MAXN+5];

int main()
{
    cin>>n;
    for (int i=1;i<=n;i++)
    {
        cin>>a[i];
        if (i>=1)
        {
            sum[i]=a[i]^sum[i-1];
        }
    }
    long long ans=0;
    for (int l=1;l<=n;l++)
    {
        for (int r=l;r<=n;r++)
        {
            ans+=sum[r]^sum[l-1];
        }
    }
    cout<<ans;
    return 0;
}
```

注意 `ans` 需要开 `long long`

常规做法到这里就是极限了。如果要继续优化，就需要从异或运算的性质入手。位运算中，每个位的运算结果只与当前位有关，所以可以使用“拆位”的技巧，将位拆出来单独处理。注意到

```cpp
    for (int l=1;l<=n;l++)
    {
        for (int r=l;r<=n;r++)
        {
            ans+=sum[r]^sum[l-1];
        }
    }
```

这段代码实际上是将 `sum` 数组中的元素两两异或然后求和。每个元素恰好都与其它元素异或 $1$ 次。求和运算中，只有异或之后值为 $1$ 的位对求和运算有影响，表现为异或前两个操作数在该位上相异。因此可以这样考虑：使用一个新的数组 `w[][2]` 来统计每个位上 $0$ 和 $1$ 的数目，那么 `w[i][0]*w[i][1]` 实际上就是两两异或的过程中，该位的结果会产生的 $1$ 的个数（相应的，会产生 $0$ 的个数应该为 `w[i][0]*w[i][0]+w[i][1]*w[i][1]`）。这个数再乘上当前位的位权，就是它最终对 `ans` 做的贡献。也就是说，$ans=\sum_{i=0}^{20}{w[i][0]\times w[i][1]\times 2^i}$ 这里累加下标从 $0$ 开始，是因为当 `l=0` 时，`sum[0]` 会参与异或。而上界 $20$ 是根据数据范围。

由此很容易得到最终代码。注意循环范围和求和中的溢出问题。

## 代码

附上通过代码：

```cpp
#include <array>
#include <iostream>
using namespace std;
const int MAXN = 100000;
int n;
array<int, MAXN + 5> sum, a;
int w[25][2];

int main()
{
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> a[i];
        sum[i] = a[i] ^ sum[i - 1];
    }
    long long ans = 0;
    for (int i = 0; i <= n; i++)
    {
        for (int j = 0; j <= 20; j++)
        {
            w[j][(sum[i]>>j)&1]++;
        }
    }
    for (int i = 0; i <= 20; i++)
    {
        ans += (long long)w[i][0] * w[i][1] * (1 << i);
    }
    cout << ans;
    return 0;
}
```