---
title: "[leetcode]搜索旋转排序数组"
date: 2025-03-21T16:12:45+08:00
categories:
    - 算法
tags:
    - 二分
    - leetcode
image: images/cover.webp
---

封面图源：[https://www.pixiv.net/artworks/120566071](https://www.pixiv.net/artworks/120566071)

## 题目描述

[搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/description/)

整数数组 nums 按升序排列，数组中的值 互不相同 。

在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为 [4,5,6,7,0,1,2] 。

给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回 -1 。

你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。

## 解题思路

又是一道二分题……但这次我做出来了！

这道题看到时间复杂度 $O(\log n)$ 就知道肯定只能用二分。但是，这道题给出的数组并不是有序的。怎么办呢？注意到题目中给出的数组是一个旋转后的有序数组，那么我们只要知道旋转点 `k`，就能将数组拆成两个有序数组，在这两个有序数组里面分别搜索即可。那么怎么找到旋转点 `k` 呢？很显然也只能用二分。注意到旋转点 `k` 左侧的所有数都 `>= nums[0]`，右侧的所有数都 `< nums[0]`，我们可以用 `nums[0]` 做二分条件，对于二分的`l, r, mid`，如果 `nums[mid]>=nums[0]`，说明 `mid` 偏左，令 `l=mid+1`；否则说明 `mid` 偏右，令 `r=mid`。那么什么时候 `mid` 刚刚好呢？当 `nums[mid]>nums[mid+1]` 时，`mid` 就刚刚好了。找到 `mid` 之后，分别在 $[first,mid]$ 和 $(mid,last)$ 内进行二分搜索 `target` 即可。

## 代码

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        if (nums.size() == 1) {
            return nums[0] == target ? 0 : -1;
        }
        auto a = nums[0];
        size_t l = 0, r = nums.size() - 1, mid;
        auto flag = false;
        while (l < r) {
            mid = (l + r) / 2;
            if (nums[mid] > nums[mid + 1]) {
                flag = true;
                break;
            } else {
                if (nums[mid] >= a)
                    l = mid + 1;
                else
                    r = mid;
            }
        }
        if (!flag)
        {
            auto result=lower_bound(nums.begin(),nums.end(),target);
            if (result==nums.end()||*result!=target) return -1;
            else return result-nums.begin();
        }else{
            auto midit=nums.begin()+mid+1;
            auto result=lower_bound(nums.begin(),midit,target);
            if (result!=midit&&*result==target) return result-nums.begin();
            else{
                result=lower_bound(midit, nums.end(),target);
                if (result!=nums.end()&&*result==target) return result-nums.begin();
                else return -1;
            }
        }
    }
};
```