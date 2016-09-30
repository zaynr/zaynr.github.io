---
title: 'LeetCode 解题报告: 174. Dungeon Game'
date: 2016-09-30 17:35:03
tags: [LeetCode]
category: [解题报告]
---
[174. Dungeon Game](https://leetcode.com/problems/dungeon-game/)
> The demons had captured the princess (P) and imprisoned her in the bottom-right corner of a dungeon. The dungeon consists of M x N rooms laid out in a 2D grid. Our valiant knight (K) was initially positioned in the top-left room and must fight his way through the dungeon to rescue the princess.
> 
> The knight has an initial health point represented by a positive integer. If at any point his health point drops to 0 or below, he dies immediately.
> 
> Some of the rooms are guarded by demons, so the knight loses health (negative integers) upon entering these rooms; other rooms are either empty (0's) or contain magic orbs that increase the knight's health (positive integers).
> 
> In order to reach the princess as quickly as possible, the knight decides to move only rightward or downward in each step.
> 
> Write a function to determine the knight's minimum initial health so that he is able to rescue the princess.
> 
> For example, given the dungeon below, the initial health of the knight must be at least 7 if he follows the optimal path RIGHT-> RIGHT -> DOWN -> DOWN.
> 
> -2 (K)	 -3	        3
> -5	    -10	        1
> 10	     30	   -5 (P)
> 
> Notes:
> 
> The knight's health has no upper bound.
> Any room can contain threats or power-ups, even the first room the knight enters and the bottom-right room where the princess is imprisoned.

<!--more-->

这道题其实也是类似于动态规划, 弄个对应于迷宫的 minHealth 数组就行了, 迷宫的每一格对应于 minHealth 数组的每个下标位置. 首先把 minHealth 初始化为与所给迷宫数组相对应 `size` 的数组, 其中每一个元素初始值赋为 1. 然后从 `Princess` 所在的地方开始遍历, 若 `Princess` 所在的 `dungeon[i][j]` 值小于零, 则需要 `minHeal[i][j] -= dungeon[i][j]`, 不然 `knight` 就死了嘛. 因为 `knight` 只能 `down` or `right`, 所以首先遍历最下边的一行和最右边的的一列. 最优化选择根据当前 `dungeon[i][j]` 的值进行判断. minVal 为可选的下个路径点之间得出的最小值.
* dungeon[i][j] > 0
    * 这里是加血, 所以只需要判断能不能加到下个地方所需要的最小 health, 若不能则需要修改 `minHeal[i][j] = minVal - dungeon[i][j]`. 判断条件为 `minVal > dungeon[i][j]`.
* dungeon[i][j] == 0
    * 这里基本可以忽略, 其实就是一个过渡格子, `minHeal[i][j] = minVal`. 
* dungeon[i][j] < 0
    * 这里会扣血, 所以就直接往上加了, `minHeal[i][j] = minVal - dungeon[i][j]`.

到这里其实就差不多了, 剩下的都是一些小细节, 上代码:
```cpp
class Solution {
public:
    int min(int a, int b){
        return a<b?a:b;
    }

    int calculateMinimumHP(vector<vector<int>>& dungeon) {
        vector<vector<int>> minHeal;
        int m = dungeon.size();
        int n = dungeon[0].size();
        for(int i = 0; i < m; i++){
            vector<int> temp;
            for(int j = 0; j < n; j++){
                temp.push_back(1);
            }
            minHeal.push_back(temp);
        }
        if(dungeon[m - 1][n - 1] < 0){
            minHeal[m - 1][n - 1] -= dungeon[m - 1][n - 1];
        }
        //buttom row
        for(int j = n - 2; j > -1; j--){
            if(dungeon[m - 1][j] > 0){
                if(minHeal[m - 1][j + 1] > dungeon[m - 1][j]){
                    minHeal[m - 1][j] = minHeal[m - 1][j + 1] - dungeon[m - 1][j];
                }
            }
            else if(dungeon[m - 1][j] == 0){
                minHeal[m - 1][j] = minHeal[m - 1][j + 1];
            }
            else{
                minHeal[m - 1][j] = minHeal[m - 1][j + 1] - dungeon[m - 1][j];
            }
        }
        //rightest row
        for(int i = m - 2; i > -1; i--){
            if(dungeon[i][n - 1] > 0){
                if(minHeal[i + 1][n - 1] > dungeon[i][n - 1]){
                    minHeal[i][n - 1] = minHeal[i + 1][n - 1] - dungeon[i][n - 1];
                }
            }
            else if(dungeon[i][n - 1] == 0){
                minHeal[i][n - 1] = minHeal[i + 1][n - 1];
            }
            else{
                minHeal[i][n - 1] = minHeal[i + 1][n - 1] - dungeon[i][n - 1];
            }
        }
        //inside the mat, except buttom line and rightest line
        for(int i = m - 2; i > -1; i--){
            for(int j = n - 2; j > -1; j--){
                int minVal = min(minHeal[i + 1][j], minHeal[i][j + 1]);
                if(dungeon[i][j] > 0){
                    if(minVal > dungeon[i][j]){
                        minHeal[i][j] = minVal - dungeon[i][j];
                    }
                }
                else if(dungeon[i][j] == 0){
                    minHeal[i][j] = min(minHeal[i + 1][j], minHeal[i][j + 1]);
                }
                else{
                    minHeal[i][j] = minVal - dungeon[i][j];
                }
            }
        }
        return minHeal[0][0];
    }
};
```