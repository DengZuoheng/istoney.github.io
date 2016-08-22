---
layout: post
title: LeetCode Problems - Linked List
category: Job Hunting
tags: [job, algorithm, leetcode]
---
{% include JB/setup %}

### 19 - Remove Nth Node from End of List

```c++
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
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        ListNode* h = new ListNode(0);
        h->next = head;

        ListNode* first = h;
        for(int i=0;i<n && first!=NULL;++i)
            first = first->next;

        ListNode* second = h;
        while(first->next!=NULL) {
            first = first->next;
            second = second->next;
        }

        ListNode* toBeDelete = second->next;
        second->next = toBeDelete->next;
        delete toBeDelete;

        return h->next;
    }
};
```

### 234 - Palindrome Linked ListNode

```c++
```
