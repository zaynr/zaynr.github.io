---
title: 'LeetCode 解题报告: 23. Merge k Sorted Lists'
date: 2016-09-29 21:18:21
tags: [LeetCode]
category: [解题报告]
---
> Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

<!--more-->

这道题很容易超时. 我对本题的第一反应: 不就是 Merge Sort 的实现吗... 但是这个数据结构是链表数组, 也就是说, 如果不把已经排序过的空链表 erase 掉, 会出错, 而每次 erase 操作的时间复杂度是 O(n), 每次 merge 因为是从 lists 里找最小值, 时间复杂度也是 O(n), 即总的时间复杂度为 O(n²). 超时.

另一个思路是, 首先对 lists 根据头结点的 val 进行逆序排序, 时间复杂度为 O(n * lgn). 之后直接取 lists[lists.size() - 1] 的头结点插入链表, 若取过之后为空则直接 lists.pop_back(), 时间复杂度为 O(1), 若取过之后不为空, 则重新调用排序算法对 lists 进行逆序排序. 最坏情况下的时间复杂度为 O(n² * k * lgn), k 为链表平均长度, 理想情况下时间复杂度为 O(n * lgn), 即每个链表只有一个结点. 所以该算法虽然在 LeetCode 被 ac 了, 但是在最坏情况下也会超时.

还有一个思路是建小顶堆, 然后每次取堆顶, 再移动被取的链表指针, 重新建堆. 此时的时间复杂度最坏情况其实也是 O(n² * k * lgn).

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