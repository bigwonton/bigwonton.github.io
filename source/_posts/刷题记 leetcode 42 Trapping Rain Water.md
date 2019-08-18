---
title: Leetcode 42
tags: 
- leetcode
- 算法
toc: true
---

## 42. Trapping Rain Water

> Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

![image](https://assets.leetcode.com/uploads/2018/10/22/rainwatertrap.png)

Example:
```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

<!--more-->


## 分析

这题目看着不难，用栈来做就可以。我的思路是：
1. 以最大值为划分，分别计算左边和右边的面积和，然后相加
2. 减去数组的每个值之和
3. 加上最大值（划分点）的值（因为这个面积没有计算在第1步的和中，第2步却减掉了，所以要加回来）

整体借助了两个stack，栈顶记录着当前位置最大的值。比栈顶大，才入栈，入栈的同时计算到前一个入栈的位置之间包含的面积。

```java
class Solution {
    public int trap(int[] height) {
         if (height == null || height.length < 1) {
            return 0;
        }
        int res = 0;
        LinkedList<Integer> lstack = new LinkedList<>();
        LinkedList<Integer> rstack = new LinkedList<>();
        int i = 0;
        for (i = 0; i < height.length; i++) {
            if (!lstack.isEmpty() && height[lstack.peek()] > height[i]) {

            } else {
                if (!lstack.isEmpty()) {
                    res += (height[lstack.peek()] * (i - lstack.peek())) ;
                }
                lstack.push(i);
            }
        }
         for (i = height.length - 1; i >= lstack.peek(); i--) {
            if (!rstack.isEmpty() && height[rstack.peek()] > height[i]) {

            } else {
                if (!rstack.isEmpty()) {
                    res += (height[rstack.peek()]) * (rstack.peek() - i);
                }
                rstack.push(i);
            }
        }
        for (i = 0; i < height.length; i++) {
            res -= height[i];
        }
        res += height[lstack.peek()];

        // System.out.println(res);
        return res;
    }
}
```

整个过程中我反复提交了好几次，才通过，犯了以下几个错误：
1. 没有考虑边界
2. 从右到左的范围选定错误，导致有一段区域在有些情况重复计算。正确的应该选择从[lstack.peek(),height.length-1]的范围