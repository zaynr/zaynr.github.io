---
title: 'LeetCode 解题报告: 45. Jump Game II'
date: 2016-09-29 19:03:36
tags: [LeetCode]
category: [解题报告]
---
> Given an array of non-negative integers, you are initially positioned at the first index of the array.
> 
> Each element in the array represents your maximum jump length at that position.
> 
> Your goal is to reach the last index in the minimum number of jumps.
> 
> For example:
> Given array A = [2,3,1,1,4]
> 
> The minimum number of jumps to reach the last index is 2. (Jump 1 step from index 0 to 1, then 3 steps to the last index.)
> 
> Note:
> You can assume that you can always reach the last index.

该题我本来想用类似于动态规划的方法来解, 即令创建一个 Count 数组来存储从数组开头移动到对应 index 的步数, 初始值赋为 `INT_MAX`, 从头开始遍历, 范围是 `[index, index + array[index] > array.size() ? index + array[index]:array.size()]`. 时间复杂度最坏情况下为 O(n²), 自然超时了.

后来机智的我想到了, 既然 index 位的数字已经赋予之后位的数字步数了, 如果已经被赋值之后位的数字的最大步长仍然无法'接触'到未赋值的的位, 则必然比原来的步长小. 类似于 `[index....[index2 ...]..]` 这样的包含状态, 可以很明显地看出, 若对 index2 进行步数遍历, index2 范围内的步数必然大于 index 范围内的步数. 所以此时可以直接 break, 不用管 index2 范围内的了. 此时将 Count 数组初始值赋为 0, 若 `Count[index + (index + array[index] < array.size() ? array[index]:(array.size() - index - 1))] != 0`, 则表明此时 index 被包含在了之前的已经遍历过了的范围内, 直接跳出循环, 进行下位的范围遍历.

以下为代码:
```cpp
class Solution {
public:
    
    int jump(vector<int>& nums) {
        vector<int> count;
        for(int i=0; i<nums.size();i++){
            count.push_back(0);
        }
        for(int i=0; i<nums.size();i++){
            int bound = (nums[i] + i) < nums.size() ? nums[i]:(nums.size() - i - 1);
            for(int j=bound; j > 0; j--){
                if(count[j + i] != 0){
                    break;
                }
                    count[j + i] = count[i] + 1;
            }
        }
        return count[nums.size() - 1];
    }
};
```

It's easy, huh.