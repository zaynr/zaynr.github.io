---
title: 'LeetCode 解题报告: 62. Unique Paths'
date: 2016-09-30 09:09:59
tags: [LeetCode]
category: [解题报告]
---
[62. Unique Paths](https://leetcode.com/problems/unique-paths/)
> A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).
> 
> The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).
> 
> How many possible unique paths are there?

> Note: m and n will be at most 100.

这道题挺简单的, 因为机器人被限定只能向左或向右移动, 并且 m 和 n 的最大值也被限定了, 所以可以直接用动态规划来做. 建立数组 mat 来记录每个点到终点的 'unique path' 数. 比如 Oj 里给的范例是 3 * 7, 那么第 7 列所有的格子因为只能向下移动, 显然 'unique path' = 1. 同理, 第三行所有格子也只能向右移动, 'unique path' = 1. 而其他的格子有两种选择, 要么向右要么向下, 即将右边格子和下边格子的 'unique path' 加起来就行了.

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        int mat[100][100];
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                mat[i][j] = 1;
            }
        }
        for(int i = m - 2; i > -1; i--){
            for(int j = n - 2; j > -1; j--){
                mat[i][j] = mat[i+1][j] + mat[i][j+1];
            }
        }
        return mat[0][0];
    }
};
```