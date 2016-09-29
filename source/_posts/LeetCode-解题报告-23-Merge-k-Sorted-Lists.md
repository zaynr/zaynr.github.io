---
title: 'LeetCode 解题报告: 23. Merge k Sorted Lists'
date: 2016-09-29 21:18:21
tags: [LeetCode]
category: [解题报告]
---
> Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

<!--more-->

思路晚点再填坑, 先回家~

```cpp
#include<algorithm>
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    static bool compare(ListNode* a, ListNode* b){
        return a->val > b->val;
    }

    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode *res, *resHead;
        resHead = res = NULL;
        for(int i = 0; i < lists.size(); i++){
            if(lists[i]==NULL){
                vector<ListNode*>::iterator it = lists.begin();
                lists.erase(it + i);
                i = -1;
            }
        }
        if(0 == lists.size()){
            return resHead;
        }
        sort(lists.begin(), lists.end(), compare);
        resHead = res = new ListNode(lists[lists.size() - 1]->val);
        lists[lists.size() - 1] = lists[lists.size() - 1]->next;
        if(lists[lists.size() - 1] == NULL){
            lists.pop_back();
        }
        else{
            sort(lists.begin(), lists.end(), compare);
        }
        while(true){
            if(0 == lists.size()){
                break;
            }
            if(lists[lists.size() - 1] != NULL){
                ListNode *temp = new ListNode(lists[lists.size() - 1]->val);
                res->next = temp;
                res = res->next;
                lists[lists.size() - 1] = lists[lists.size() - 1]->next;
            }
            if(lists[lists.size() - 1] == NULL){
                lists.pop_back();
            }
            else{
                sort(lists.begin(), lists.end(), compare);
            }
        }
        return resHead;
    }
};
```