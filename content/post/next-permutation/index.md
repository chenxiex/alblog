---
title: "下一个排列"
date: 2025-03-21T14:49:55+08:00
categories:
    - 算法
tags:
    - 数学
image: images/cover.webp
---

封面图源 [https://www.pixiv.net/artworks/128286318](https://www.pixiv.net/artworks/128286318)

在 leetcode 上看到一道题[下一个排列](https://leetcode.cn/problems/next-permutation/description/)，很明显就是 C++ 中的 std::next_permutation。借此机会研究一下 C++ 中 std::next_permutation 的实现。

## 示例代码

```cpp
template<class BidirIt>
bool next_permutation(BidirIt first, BidirIt last)
{
    auto r_first = std::make_reverse_iterator(last);
    auto r_last = std::make_reverse_iterator(first);
    auto left = std::is_sorted_until(r_first, r_last);
 
    if (left != r_last)
    {
        auto right = std::upper_bound(r_first, left, *left);
        std::iter_swap(left, right);
    }
 
    std::reverse(left.base(), last);
    return left != r_last;
}
```

## 算法解释

一般求排列，大多数都是求全排列，时间复杂度和空间复杂度都是 O(n!)。此时只需要枚举每个位上的数字即可。但是，对于求下一个排列，这样做效率太低了。std::next_permutation 可以做到在 O(n) 的时间复杂度下原地操作求出下一个排列。该算法乍看不好理解，实际上很像求一个二进制数+1 的算法。下面解释为什么。

首先看 `auto left = std::is_sorted_until(r_first, r_last);` 这一句。这一句是求从后往前的最长不下降序列。也就是说，求 `left`，使得区间 $(left, last)$ 内的元素是降序的。为什么要求这个呢？我们从两点来思考。首先，要求字典序上的“下一个”，必然是从后往前更改的。其次，对于一个排列，如果它是降序的，那么它就是字典序上最大的排列了。这就有点像求 $10101111$，首先要找到最后的 $1111$。$1111$ 就是 4 位数里面最大的了，如果再 +1，就务必要进位，那么在整个 +1 的过程中，就只有最后的 5 位 $01111$ 会发生变化，前面的 $101$ 我们就可以不管了。放在求下一个排列也是一样的道理，在求下一个排列的过程中，只有区间 $[left, last)$ 内的数可能发生变化，`left` 前面的数是不用管的。

那么 $[left, last)$ 内的数要怎么变化呢？$01111+1=10000$，从这个式子我们可以看出，求 $01111+1$ 实际上就是把最前面 $0+1$，然后把后面的 $1111$ 变成最小的，在二进制里面就是 $0000$。那么在求下一个排列里面的？就是在 $(left, right)$ 中找到最小的比 `*left` 大的数，把 `*left`和它交换，这个有点像求 $0+1$。在上面的示例代码中就是 `if (left != r_last)` 这一段的操作。然后需要把后面的 $(left, right)$ 变成最小的，升序排列就是最小的，因此这里可以用 `std::sort(left.base(), last);`。读者可以试试用 `std::sort(left.base(), last)` 代替 `std::reverse(left.base(), last)`，最终结果应该也是对的。那为什么这里用了 `std::reverse(left.base(), last)` 呢？注意到我们前面提到 $(left, last)$ 内的元素是降序的，刚刚的交换也不会破坏这个有序性，因此在交换完之后 $(left, last)$ 依然是降序排列的。那么就不需要用 `std::sort()` 了，直接用 `std::reverse()` 将降序的序列翻转一下就可以变成升序了。