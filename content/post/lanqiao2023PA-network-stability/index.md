---
title: "[蓝桥杯2023省A]网络稳定性"
date: 2025-01-18T11:01:57+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - 最近公共祖先 LCA
    - 生成树
image: images/cover.webp
---

封面图：[https://www.pixiv.net/artworks/126215041](https://www.pixiv.net/artworks/126215041)

## 题目描述

[洛谷P9235](https://www.luogu.com.cn/problem/P9235)

有一个局域网，由 $n$ 个设备和 $m$ 条物理连接组成，第 $i$ 条连接的稳定性为 $w_i$。

对于从设备 $A$ 到设备 $B$ 的一条经过了若干个物理连接的路径，我们记这条路径的稳定性为其经过所有连接中稳定性最低的那个。

我们记设备 $A$ 到设备 $B$ 之间通信的稳定性为 $A$ 至 $B$ 的所有可行路径的稳定性中最高的那一条。

给定局域网中的设备的物理连接情况，求出若干组设备 $x_i$ 和 $y_i$ 之间的通信稳定性。如果两台设备之间不存在任何路径，请输出 $-1$。

### 输入格式

输入的第一行包含三个整数 $n,m,q$，分别表示设备数、物理连接数和询问数。

接下来 $m$ 行，每行包含三个整数 $u_i,v_i,w_i$，分别表示 $u_i$ 和 $v_i$ 之间有一条稳定性为 $w_i$ 的物理连接。

接下来 $q$ 行，每行包含两个整数 $x_i,y_i$，表示查询 $x_i$ 和 $y_i$ 之间的通信稳定性。

### 输出格式

输出 $q$ 行，每行包含一个整数依次表示每个询问的答案。

### 样例 #1

#### 样例输入 #1

```
5 4 3
1 2 5
2 3 6
3 4 1
1 4 3
1 5
2 4
1 3
```

#### 样例输出 #1

```
-1
3
5
```

### 提示

【评测用例规模与约定】

对于 $30 \%$ 的评测用例，$n,q \leq 500$，$m \leq 1000$；

对于 $60 \%$ 的评测用例，$n,q \leq 5000$，$m \leq 10000$；

对于所有评测用例，$2 \leq n,q \leq 10^5$，$1 \leq m \leq 3 \times 10^5$，$1 \leq u_i,v_i,x_i,y_i \leq n$，$
1 \leq w_i \leq 10^6$，$u_i \neq v_i$，$x_i \neq y_i$。

## 解题思路

这道题最开始很容易往最短路的方向想，但是最短路的复杂度是 $O(n^3)$，对于 $n \leq 10^5$ 的数据量是不可接受的。

注意到路径的稳定性与路径中经过的单条边的边权有关，但与整条路径的长度无关。因此其实可以对每条边做贪心，而不需要像最短路一样计算整条路径的长度。又因为两个设备之间的稳定性为所有可行路径中稳定性最高的一条，因此贪心的策略就是要不断取稳定性高的边，直到设备之间连通起来。这实际上是“最大生成树”。其算法可以采用和最小生成树一样的算法。这样处理之后，我们就可以剪去很多无用的路径，从图中生成几颗“最大生成树”。在同一颗生成树内的就是互相连通的，否则不能到达。这可以通过求生成树过程中的并查集来判断。

生成了树之后，就要考虑如何高效地处理查询。尽管经过剪枝，图的路径数已经大大降低了，但每个查询跑一遍搜索大概还是不能接受的。考虑到这是一棵树，可以使用最近公共祖先算法，让两个叶子节点同时向根部回溯直到相遇，记录下回溯过程中经过的最小边权即为答案。

具体实现上，最小生成树算法并不能实际给到一个树的数据结构，只能得到一张有向无环图。可以在算法过程中处理来得到树，我这里直接选择在有向无环图上跑一遍dfs来生成树，时间复杂度 $O(n)$ 。朴素 LCA 的时间复杂度是 $O(n)$ ，每次查询都跑一遍的话最后一个点会超时，需要使用倍增 LCA，倍增 LCA 需要注意倍增的上界，避免数组越界问题。

## 代码

最后附上代码：

```cpp
#include <algorithm>
#include <cassert>
#include <cmath>
#include <iostream>
#include <list>
#include <queue>
#include <utility>
#include <vector>
using namespace std;
const int MAXN = 100000, MAXQ = 100000, MAXM = 300000, MAXW = 1000000;
struct edge
{
    int u, v, w;
    bool operator<(const edge &other) const
    {
        return w < other.w;
    }
    bool operator>(const edge &other) const
    {
        return w > other.w;
    }
    bool operator==(const edge &other) const
    {
        return w == other.w;
    }
};
struct ledge
{
    int to, w;
};
struct lnode
{
    list<ledge> e;
};
struct tnode
{
    vector<ledge> fa;
    list<ledge> son;
    int d = -1;
} tree[MAXN + 5];
class ufset
{
    struct ufnode
    {
        int f = -1, d = 1;
    };
    vector<ufnode> node;

  public:
    ufset(int n)
    {
        node.resize(n);
    }
    int find(int a)
    {
        if (node[a].f == -1)
            return a;
        else
            return node[a].f = find(node[a].f);
    }
    int uni(int a, int b)
    {
        a = find(a);
        b = find(b);
        if (a == b)
            return -1;
        if (node[a].d < node[b].d)
            node[a].f = b;
        else
        {
            if (node[a].d == node[b].d)
            {
                node[a].f = b;
                node[b].d++;
            }
            else
            {
                node[b].f = a;
            }
        }
        return 0;
    }
} uf(MAXN + 5);
int n, m, q;

void dfs(int cur, int fa, int d, const vector<lnode> &ln)
{
    tree[cur].d = d;
    for (auto i:ln[cur].e)
    {
        if (fa != -1 && i.to == fa)
        {
            tree[cur].fa.push_back(i);
        }
        else
        {
            tree[cur].son.push_back(i);
        }
    }
    for (int i = 0; i < tree[cur].fa.size(); i++)
    {
        int fa = tree[cur].fa[i].to;
        if (tree[fa].fa.size() <= i)
            break;
        tree[cur].fa.push_back({tree[fa].fa[i].to, min(tree[fa].fa[i].w, tree[cur].fa[i].w)});
    }
    for (auto i : tree[cur].son)
    {
        dfs(i.to, cur, d + 1, ln);
    }
}
void gen_tree()
{
    priority_queue<edge> e;
    for (int i = 0; i < m; i++)
    {
        edge ei;
        cin >> ei.u >> ei.v >> ei.w;
        e.push(std::move(ei));
    }
    int sum = 0;
    vector<lnode> ln(n + 5);
    while (sum < n - 1 && !e.empty())
    {
        edge ei = e.top();
        e.pop();
        if (uf.uni(ei.u, ei.v) == -1)
            continue;
        sum++;
        ln[ei.u].e.push_back({ei.v, ei.w});
        ln[ei.v].e.push_back({ei.u, ei.w});
    }
    for (int i = 1; i <= n; i++)
    {
        if (tree[i].d == -1)
        {
            dfs(i, -1, 0, ln);
        }
    }
}
int lca(int u, int v)
{
    int ans = MAXW + 5;
    while (tree[u].d > tree[v].d)
    {
        int d = log2(tree[u].d - tree[v].d);
        ans = min(tree[u].fa[d].w, ans);
        u = tree[u].fa[d].to;
    }
    while (tree[v].d > tree[u].d)
    {
        int d = log2(tree[v].d - tree[u].d);
        ans = min(tree[v].fa[d].w, ans);
        v = tree[v].fa[d].to;
    }
    assert(tree[u].d == tree[v].d);
    if (u == v)
        return ans;
    while (tree[u].fa[0].to != tree[v].fa[0].to)
    {
        int logd = log2(tree[u].d);
        for (int i = logd; i >= 0; i--)
        {
            if (tree[u].fa[i].to == tree[v].fa[i].to)
                continue;
            ans = min(min(ans, tree[u].fa[i].w), tree[v].fa[i].w);
            u = tree[u].fa[i].to;
            v = tree[v].fa[i].to;
            break;
        }
    }
    ans = min(min(ans, tree[u].fa[0].w), tree[v].fa[0].w);
    return ans;
}
void ans()
{
    for (int i = 0; i < q; i++)
    {
        int u, v;
        cin >> u >> v;
        if (uf.find(u) != uf.find(v))
        {
            cout << -1 << endl;
            continue;
        }
        cout << lca(u, v) << endl;
    }
}

int main()
{
    cin >> n >> m >> q;
    gen_tree();
    ans();
    return 0;
}
```