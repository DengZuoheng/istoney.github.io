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

### 21 - Merge Two Sorted Lists

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
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode head(0);
        ListNode * cur = &head;

        ListNode *i = l1, *j = l2;
        while(i != NULL && j != NULL) {
            if(i->val < j->val) {
                cur->next = i;
                i = i->next;
            }
            else {
                cur->next = j;
                j = j->next;
            }
            cur = cur->next;
        }

        if(i == NULL) {
            cur->next = j;
        }
        else {
            cur->next = i;
        }

        return head.next;
    }
};
```

### 23 - Merge k Sorted Lists

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
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        ListNode head(0);
        ListNode *cur = &head;

        priority_queue<ListNode*, vector<ListNode*>, cmp> que;

        for(int i=0;i<lists.size();++i) {
            if(lists[i] != NULL)
                que.push(lists[i]);
        }

        ListNode * tmp;
        while(!que.empty()) {
            tmp = que.top();
            que.pop();
            cur->next = tmp;
            cur = cur->next;

            if(tmp->next != NULL)
                que.push(tmp->next);
        }

        cur->next = NULL;

        return head.next;
    }

    struct cmp {
        bool operator()(ListNode *l1, ListNode *l2) {
            return l1->val > l2->val;
        }
    };
};
```

### 24 - Swap Nodes in Pairs

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
    ListNode* swapPairs(ListNode* head) {
        ListNode empty(0);
        empty.next = head;

        ListNode *cur = &empty;
        ListNode *first, *second;

        while(cur != NULL) {
            first = cur->next;
            if(first == NULL) break;
            second = first->next;
            if(second == NULL) break;

            cur->next = second;
            first->next = second->next;
            second->next = first;

            cur = first;
        }

        return empty.next;
    }
};
```

### 25 - Reverse Nodes in k-Groups

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
    ListNode* reverseKGroup(ListNode* head, int k) {
        if(head == NULL || k <= 1) return head;

        ListNode empty(0);
        ListNode* tail = &empty;
        tail->next = head;

        while(true) {
            int c = 0;
            ListNode* nextHead = head;
            while(nextHead != NULL && c < k) {
                nextHead = nextHead->next;
                ++c;
            }

            if(c < k) break;

            ListNode* newTail = head;
            ListNode* toInsert = head->next;

            for(int i=0;i<k-1;++i) {
                ListNode* tmp = toInsert->next;
                toInsert->next = tail->next;
                tail->next = toInsert;
                toInsert = tmp;
            }

            tail = newTail;
            head = nextHead;
            tail->next = head;
        }

        return empty.next;
    }
};
```
