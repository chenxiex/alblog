---
title: "[蓝桥杯2023国A]01游戏题解"
date: 2024-08-20T21:09:39+08:00
categories:
    - 算法
tags:
    - 蓝桥杯
    - DFS
image: images/cover.webp
---
封面图：[https://www.bilibili.com/opus/967606108335112258](https://www.bilibili.com/opus/967606108335112258)
## 题目描述
[洛谷P10419](https://www.luogu.com.cn/problem/P10419)

小蓝最近玩上了 $01$ 游戏，这是一款带有二进制思想的棋子游戏，具体来说游戏在一个大小为 $N\times N$ 的棋盘上进行，棋盘上每个位置都需要放置一位数字 $0$ 或者数字 $1$，初始情况下，棋盘上有一部分位置已经被放置好了固定的数字，玩家不可以再进行更改。玩家需要在其他所有的空白位置放置数字，并使得最终结果满足以下条件：

1. 所有的空白位置都需要放置一个数字 $0/1$；
2. 在水平或者垂直方向上，相同的数字不可以连续出现大于两次；
3. 每一行和每一列上，数字 $0$ 和数字 $1$ 的数量必须是相等的 (例如 $N=4$，则表示每一行/列中都需要有 $2$ 个 $0$ 和 $2$ 个 $1$)；
4. 每一行都是唯一的，因此每一行都不会和另一行完全相同；同理每一列也都是唯一的，每一列都不会和另一列完全相同。

现在请你和小蓝一起解决 $01$ 游戏吧！题目保证所有的测试数据都拥有一个唯一的答案。

### 输入格式

输入的第一行包含一个整数 $N$ 表示棋盘大小。

接下来 $N$ 行每行包含 $N$ 个字符，字符只可能是 `0`、`1`、`_` 中的其中一个 (ASCII 码分别为 $48$，$49$，$95$)，`0` 表示这个位置数字固定为 $0$，`1` 表示这个位置数字固定为 $1$，`_` 表示这是一个空白位置，由玩家填充。

### 输出格式

输出 $N$ 行每行包含 $N$ 个字符表示题目的解，其中的字符只能是 `0` 或者 `1`。

### 样例 #1

#### 样例输入 #1

```
6
_0____
____01
__1__1
__1_0_
______
__1___
```

#### 样例输出 #1

```
100110
010101
001011
101100
110010
011001
```

### 提示

**【评测用例规模与约定】**

对于 $60\%$ 的评测用例，$2\le N\le 6$;  
对于所有评测用例，$2\le N\le 10$，$N$ 为偶数。

感谢 @rui_er 提供测试数据。

## 解题思路
这道题的题目数据比较弱，虽然生成二进制数然后按行填入的方法比较快，但是用一位一位填数的方法也能过洛谷的评测。

剪枝技巧非常简单：
1. 记录每一行和每一列 $0$ 和 $1$ 的个数，仅当有余量的时候允许填 $0$ 或者填 $1$。
2. 在每行填完的时候进行合法性检查：
     1. 检查当前行是否有连续3个 $0$ 或 $1$；
     2. 检查是否有行重复；
     3. 检查已经填入的数字中，是否有某一列出现了重复的 $0$ 或 $1$。

我最开始实现的时候为了追求更快的剪枝，还希望在每填一个数的时候，通过检查该位置前后的数字，来确保该位置只填入合法的数字。比如，如果该位置前面已经有两个 $0$ 了，就直接填 $1$。但是实际实现下来这需要考虑非常多的情况，仅单个数字出现重复的情况就有6种，还要考虑到两个数字都出现重复的情况，比如 $11\\_00$。这样子判断条件就很复杂，很容易出错，而且效果不理想。经验教训就是不要考虑太过复杂，收益也不高的剪枝吧。

## 代码
最后附上代码：

```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int MAXN = 11;
struct coord
{
    int x, y;
    coord move(int n) const
    {
        if (y < n - 1)
            return {x, y + 1};
        else
            return {x + 1, 0};
    }
};
char a[MAXN][MAXN];
int r[MAXN][2], c[MAXN][2];
int n;
bool checkc()
{
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < i; j++)
        {
            bool flag = true;
            for (int k = 0; k < n; k++)
            {
                if (a[k][i] != a[k][j])
                {
                    flag = false;
                    break;
                }
            }
            if (flag)
                return false;
        }
    }
    return true;
}
bool checkr(int r)
{
    for (int k = 0; k <= r - 2; k++)
    {
        for (int i = 0; i < n; i++)
            if (a[k][i] == a[k + 1][i] && a[k][i] == a[k + 2][i])
                return false;
    }
    for (int i = 0; i < r; i++)
    {
        if (strcmp(a[i], a[r]) == 0)
            return false;
    }
    for (int j = 0; j < n - 2; j++)
    {
        if (a[r][j + 1] == a[r][j + 2] && a[r][j] == a[r][j + 1])
            return false;
    }
    return true;
}
bool dfs(coord coo)
{
    // 递归出口
    if (coo.x == n && coo.y == 0)
    {
        return checkr(coo.x-1) && checkc();
    }
    // 完成一行，进行行查重
    if (coo.y == 0 && coo.x > 0)
    {
        if (!checkr(coo.x - 1))
            return false;
    }
    // 该位置已固定
    if (a[coo.x][coo.y] != '_')
    {
        return dfs(coo.move(n));
    }
    if (r[coo.x][0]>0 && c[coo.y][0]>0)
    {
        char bak = a[coo.x][coo.y];
        a[coo.x][coo.y] = '0';
        r[coo.x][0]--;
        c[coo.y][0]--;
        if (dfs(coo.move(n)))
            return true;
        r[coo.x][0]++;
        c[coo.y][0]++;
        a[coo.x][coo.y] = bak;
    }
    if (r[coo.x][1]>0 && c[coo.y][1]>0)
    {
        char bak = a[coo.x][coo.y];
        a[coo.x][coo.y] = '1';
        r[coo.x][1]--;
        c[coo.y][1]--;
        if (dfs(coo.move(n)))
            return true;
        r[coo.x][1]++;
        c[coo.y][1]++;
        a[coo.x][coo.y] = bak;
    }
    return false;
}
int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
        r[i][0] = r[i][1] = c[i][0] = c[i][1] = n / 2;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
        {
            cin >> a[i][j];
            if (a[i][j] == '0')
            {
                r[i][0]--;
                c[j][0]--;
            }
            if (a[i][j] == '1')
            {
                r[i][1]--;
                c[j][1]--;
            }
        }
    dfs({0, 0});
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
            putchar(a[i][j]);
        putchar('\n');
    }
    return 0;
}
```