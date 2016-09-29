---
title: 'LeetCode 解题报告: 57. Insert Interval'
date: 2016-09-29 20:08:58
tags: [LeetCode]
category: [解题报告]
---
> Given a set of non-overlapping intervals, insert a new interval into the intervals (merge if necessary).
> 
> You may assume that the intervals were initially sorted according to their start times.
> 
> Example 1:
> Given intervals [1,3],[6,9], insert and merge [2,5] in as [1,5],[6,9].
> 
> Example 2:
> Given [1,2],[3,5],[6,7],[8,10],[12,16], insert and merge [4,9] in as [1,2],[3,10],[12,16].
> 
> This is because the new interval [4,9] overlaps with [3,5],[6,7],[8,10].

<!--more-->

这题其实挺弱智的... 就是考虑各种 overlap 的情况, 然后直接 push_back 进结果即可.

```cpp
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */
class Solution {
public:
    //has over lap
    //case 1: [intervals...{newInterval...]...}
    //case 2: {newInterval...[intervals...]...}
    //case 3: {newInterval...[intervals...}...]
    //case 4: [intervals...{newInterval...}...]
    vector<Interval> insert(vector<Interval>& intervals, Interval newInterval) {
        vector<Interval> res;
        Interval merge;
        if(intervals.size() == 0){
            res.push_back(newInterval);
        }
        for(int i=0; i<intervals.size(); i++){
            //case 1: [intervals...{newInterval...]...}
            //case 2: {newInterval...[intervals...]...}
            //case 3: {newInterval...[intervals...}...]
            if(intervals[i].start > newInterval.start && intervals[i].end >= newInterval.end && intervals[i].start <= newInterval.end || intervals[i].start > newInterval.start && intervals[i].end < newInterval.end || intervals[i].start <= newInterval.start && intervals[i].end >= newInterval.start && intervals[i].end < newInterval.end){
                if(intervals[i].start <= newInterval.start && intervals[i].end >= newInterval.start && intervals[i].end < newInterval.end){
                    merge.start = intervals[i].start;
                }
                else if(i==0 || (intervals[i - 1].end < newInterval.start && intervals[i].start > newInterval.start)){
                    merge.start = newInterval.start;
                }
                if(intervals[i].start > newInterval.start && intervals[i].end >= newInterval.end && intervals[i].start <= newInterval.end){
                    merge.end = intervals[i].end;
                    res.push_back(merge);
                }
                else if(i==intervals.size() - 1 || (intervals[i + 1].start > newInterval.end && intervals[i].end < newInterval.end)){
                    merge.end = newInterval.end;
                    res.push_back(merge);
                }
            }
            //{newInterval...}[intervals...]
            else if(i == 0 && intervals[i].start > newInterval.end){
                res.push_back(newInterval);
                res.push_back(intervals[i]);
            }
            //{newInterval...}[intervals...]
            else if(i == intervals.size() - 1 && intervals[i].end < newInterval.start){
                res.push_back(intervals[i]);
                res.push_back(newInterval);
            }
            //[intervals...]{newInterval...}[intervals...]
            else if(intervals[i].start > newInterval.end && intervals[i - 1].end < newInterval.start){
                res.push_back(newInterval);
                res.push_back(intervals[i]);
            }
            //case 4: [intervals...{newInterval...}...]
            else{
                res.push_back(intervals[i]);
            }
        }
        return res;
    }
};
```