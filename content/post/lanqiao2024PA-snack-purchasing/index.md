---
title: "[蓝桥杯2024省A]零食采购题解"
date: 2025-01-23T16:11:58+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 最近公共祖先 LCA
image: images/cover.webp
---

封面图源 [https://www.pixiv.net/artworks/87550307](https://www.pixiv.net/artworks/87550307)

## 题目描述
[洛谷P10391](https://www.luogu.com.cn/problem/P10391)

小蓝准备去星际旅行，出发前想在本星系采购一些零食，星系内有 $n$ 颗星球，由 $n-1$ 条航路连接为连通图，第 $i$ 颗星球卖第 $c_i$ 种零食特产。小蓝想出了 $q$ 个采购方案，第 $i$ 个方案的起点为星球 $s_i$ ，终点为星球 $t_i$ ，对于每种采购方案，小蓝将从起点走最短的航路到终点，并且可以购买所有经过的星球上的零食（包括起点终点），请计算每种采购方案最多能买多少种不同的零食。

### 输入格式

输入的第一行包含两个正整数 $n$，$q$，用一个空格分隔。  
第二行包含 $n$ 个整数 $c_1,c_2,\cdots, c_n$，相邻整数之间使用一个空格分隔。  
接下来 $n - 1$ 行，第 $i$ 行包含两个整数 $u_i,v_i$，用一个空格分隔，表示一条
航路将星球 $u_i$ 与 $v_i$ 相连。  
接下来 $q$ 行，第 $i$ 行包含两个整数 $s_i
, t_i $，用一个空格分隔，表示一个采购方案。

### 输出格式

输出 $q$ 行，每行包含一个整数，依次表示每个采购方案的答案。

### 样例 #1

#### 样例输入 #1

```
4 2
1 2 3 1
1 2
1 3
2 4
4 3
1 4
```

#### 样例输出 #1

```
3
2
```

### 提示

第一个方案路线为 $\{4, 2, 1, 3\}$，可以买到第 $1, 2, 3$ 种零食；  
第二个方案路线为 $\{1, 2, 4\}$，可以买到第 $1, 2$ 种零食。

对于 20% 的评测用例，$1 ≤ n, q ≤ 5000 $；    
对于所有评测用例，$1 ≤ n, q ≤ 10^5，1 ≤ c_i ≤ 20，1 ≤ u_i
, v_i ≤ n，1 ≤ s_i
, t_i ≤ n$。

## 解题思路

很简单的 LCA。用倍增 LCA 就可以了，时间复杂度 $O(qlogn)$。有点麻烦的是这个 $ci$ 的处理。朴素 LCA 很好处理这个 $ci$，反正是一个点一个点地回溯的，回溯的时候顺便统计 $ci$ 就好。倍增 LCA 也可以处理路径上的信息，但需要在建树的时候预处理这些信息。在祖先节点数组上记录从该节点到祖先节点（最好是半闭半开区间方便后续统计）所能够买到的所有零食种类，每次扩展祖先节点数组的时候将两个中间祖先的值合并到新祖先上即可。因为每个祖先本质上都是通过父亲扩展而来，因此并不会有遗漏（如果这点有疑问的自己模拟一遍这个过程就明白了）。这要求有一个能够方便地存储所有零食种类和执行合并操作的数据结构。我一开始使用了 `unordered_set`，用起来非常恶心。看了题解之后发现因为 $c_i$ 的范围较小，可以用 `bitset`，用了 `bitset` 马上就 AC 了。

## 代码

附上通过代码。这道题洛谷的数据终于不犯病了，洛谷和蓝桥杯官网都能过。

```cpp
#include <array>
#include <bitset>
#include <cassert>
#include <cmath>
#include <iostream>
#include <list>
#include <vector>
using namespace std;

const int MAXN = 1e5;
const int MAXCI = 20;
using bs = bitset<MAXCI + 5>;
int n, q;
struct node
{
    int c;
    list<int> v;
};
struct tnode
{
    struct father
    {
        int v;
        bs c;
    };
    vector<father> f;
    list<int> s;
    int d;
};
array<node, MAXN + 5> g;
array<tnode, MAXN + 5> tree;

void dfs(int p, int f, int d)
{
    tree.at(p).d = d;
    if (f != 0)
    {
        tree.at(p).f.push_back({f, bs(1 << g.at(p).c)});
    }
    for (auto &i : g.at(p).v)
    {
        if (i != f)
        {
            tree.at(p).s.push_back(i);
        }
    }
    for (auto i = 0; i < tree.at(p).f.size(); i++)
    {
        auto &f1 = tree.at(p).f;
        auto &f2 = tree.at(f1.at(i).v).f;
        if (f2.size() > i)
        {
            f1.push_back(f2.at(i));
            f1.rbegin()->c |= f1.at(i).c;
        }
    }
    for (auto &i : tree.at(p).s)
    {
        dfs(i, p, d + 1);
    }
}
int lca(int s, int t)
{
    bs ans(0);
    while (tree.at(s).d > tree.at(t).d)
    {
        auto logd = static_cast<int>(log2(tree.at(s).d - tree.at(t).d));
        auto &f = tree.at(s).f.at(logd);
        ans |= f.c;
        s = f.v;
    }
    while (tree.at(t).d > tree.at(s).d)
    {
        auto logd = static_cast<int>(log2(tree.at(t).d - tree.at(s).d));
        auto &f = tree.at(t).f.at(logd);
        ans |= f.c;
        t = f.v;
    }
    assert(tree.at(s).d == tree.at(t).d);
    while (s != t)
    {
        auto &f1 = tree.at(s).f;
        auto &f2 = tree.at(t).f;
        auto logd = static_cast<int>(log2(tree.at(s).d));
        for (auto i = logd; i >= 0; i--)
        {
            if (f1.at(i).v == f2.at(i).v)
                continue;
            ans |= f1.at(i).c;
            s = f1.at(i).v;
            ans |= f2.at(i).c;
            t = f2.at(i).v;
        }
        if (f1.begin()->v == f2.begin()->v)
        {
            auto i = 0;
            ans |= f1.at(i).c;
            s = f1.at(i).v;
            ans |= f2.at(i).c;
            t = f2.at(i).v;
            break;
        }
    }
    ans |= (1 << g.at(s).c);
    return ans.count();
}
int main()
{
    cin >> n >> q;
    for (auto i = 1; i <= n; i++)
    {
        int c;
        cin >> c;
        g.at(i).c = c;
    }
    for (auto i = 1; i <= n - 1; i++)
    {
        int u, v;
        cin >> u >> v;
        g.at(u).v.push_back(v);
        g.at(v).v.push_back(u);
    }
    dfs(1, 0, 0);
    for (auto i = 1; i <= q; i++)
    {
        int s, t;
        cin >> s >> t;
        cout << lca(s, t) << endl;
    }
    return 0;
}
```