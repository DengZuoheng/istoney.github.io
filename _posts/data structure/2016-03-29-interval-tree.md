---
layout: post
title: 区间树(线段树) Interval Tree
category : data structure
tags: [区间树, 线段树]
---
{% include JB/setup %}

### # 定义
**区间树** (Interval tree), 是一种二叉搜索树。它将一个区间划分成一些单元区间(即单个数据)，每个单元区间对应一个叶节点，非叶节点表示其所代表的子树对应的子区间.

区间树和线段树并不是一个概念，线段树是一个静态结构，用于数据存储查询，不能进行修改；而区间树是支持动修改和查询的数据结构，很多人将这两个概念混淆，并用线段树指代区间树。

区间树中，每个节点的子节点分别表示它的左右半区间。对于每一个非叶节点 ，设其对应区间为[a, b]，它的左子树表示的区间为[a, (a+b)/2], 右子树表示的区间为[(a+b)/2+1, b]。因此区间树是平衡二叉树, 其存储的空间复杂度是O(N) (确切说是4N-2), 操作的时间复杂度是O(logN).

区间树中，叶节点的value是数据，非叶节点的value是其对应区间的最小值。

下图给出一个以数组array构建的区间树的示例。

```c++
const int N = 6;
int array[N] = {2, 5, 1, 4, 9, 3};
```

![区间树](/attaches/interval-tree-sample.jpg)

### # 构建区间树
区间树的构建采用递归方式，当前区间是单元区间时直接赋值给节点，否则，首先递归构建左右子树，然后将左右子树的最小值赋值给当前结点。其复杂度是O(N).

此处采用数组实现区间树，itvTree[0]是根节点，itvTree[node]的左右子为itvTree[node\*2+1]和itvTree[node\*2+2].

```c++
struct ItvNode{
    int value;
} itvTree[N*2];

/**
 build interval tree
 node: node index need to build
 begin: begin of the interval corresponding to node
 end: end of the interval corresponding to node, exclusive
 */
void build(int node, int begin, int end) {
    if(begin == end-1) itvTree[node].value = array[begin];
    else{
        build(node*2+1, begin, (begin+end)/2);
        build(node*2+2, (begin+end)/2, end);
        itvTree[node].value = min(itvTree[node*2+1].value, itvTree[node*2+2].value);
    }
}
```

### # 区间查询
区间查询是指，用户输入一个区间，获取该区间的相关信息，比如区间的最大值，最小值，第k大的值等。对于本文的示例程序，是获取区间的最小值。

若查询区间与当前结点对应区间没有交叠，返回-1. 若查询区间包含在当前区间内，返回当前结点的value。否则，对子区间递归查询，并将查询结果合并返回。查询复杂度为O(logN).

递归中，将查询区间不加改变的传下去，每次查询只返回包含在查询区间内的结果即可。

```c++
/**
 query for minimum number of given interval
 node: node index to query
 begin: begin of the interval corresponding to node
 end: end of the interval corresponding to node, exclusive
 left: left bounce of query interval
 right: right bounce of query interval, exclusive
 */
int query(int node, int begin, int end, int left, int right) {
    //[begin, end) -> [left, right)
    if(left>=end || right<=begin)   //query interval not overlap with current
        return -1;

    if(left <=begin && end<=right)  //current interval is included in query interval
        return itvTree[node].value; //return node's value

    //recusive query
    int res1 = query(node*2+1, begin, (begin+end)/2, left, right);
    int res2 = query(node*2+2, (begin+end)/2, end, left, right);

    if(res1 == -1) return res2;
    if(res2 == -1) return res1;
    return min(res1, res2);
}
```

### # 节点更新 & 区间更新 & 动态维护
a. 节点更新<br>
更新一个节点的value时，除了需要更新节点值，还要递归的更新其父节点，时间复杂度为O(logN).

```c++
/**
 update a node's value
 node: root node of current sub-tree
 left: left bounce of query interval
 right: right bounce of query interval, exclusive
 index: index of array need to be updated
 value: new value
 */
void update(int node, int begin, int end, int index, int value) {
    if(begin == end-1) itvTree[node].value = value;
    else {
        int mid = (begin+end) >> 1;
        if(index < mid) update(node*2+1, begin, mid, index, value);
        else update(node*2+2, mid, end, index, value);

        //update father node
        itvTree[node].value = min(itvTree[node*2+1].value, itvTree[node*2+2].value);
    }
}
```

b.区间更新<br>
区间更新是指更新某个区间内的叶子节点的值，同时涉及到的非叶节点也需要同时更新。但是因为需要更新的节点很多，其复杂度将大于O(logN)。因此引入**延迟标记**的概念，这也是区间树的精髓。

延迟标记：每个节点增加一个标记，用于记录该节点是否进行了某种修改。当进行区间更新时，按照区间查询的方式，将更新区间划分为区间树中的节点，对这些修改这些、

```c++
struct ItvNode {
    int value;
    int mark;
} itvTree[N*2];

void build(int node, int begin, int end) {
    itvTree[node].mark = 0;
    if(begin == end-1) itvTree[node].value = array[begin];
    else{
        build(node*2+1, begin, (begin+end)/2);
        build(node*2+2, (begin+end)/2, end);
        itvTree[node].value = min(itvTree[node*2+1].value, itvTree[node*2+2].value);
    }
}

void pushDown(int node) {
    if(itvTree[node].mark != 0) {
        itvTree[node*2+1].mark += itvTree[node].mark;
        itvTree[node*2+2].mark += itvTree[node].mark;

        itvTree[node*2+1].value += itvTree[node].mark;
        itvTree[node*2+2].value += itvTree[node].mark;

        itvTree[node].mark = 0;
    }
}

int query(int node, int begin, int end, int left, int right) {
    if(left>=end || right<=begin)
        return -1;

    if(left<=begin && end<=right)   //current interval is included in query interval
        return itvTree[node].value;

    pushDown(node);

    int res1 = query(node*2+1, begin, (begin+end)/2, left, right);
    int res2 = quert(node*2+2, (begin+end)/2, end, left, right);
    if(res1 == -1) return res2;
    if(res2 == -1) return res1;
    return min(res1, res2);
}

void update(int node, int begin, int end, int left, int right, int mark) {
    if(left>=end || right<=begin) return;

    if(left<=begin && end<=right){
        itvTree[node].value += mark;    //update node's value first
        itvTree[node].mark += mark;
        return;
    }

    pushDown(node); //push previous update down

    update(node*2+1, begin, (begin+end)/2, left, right, mark);
    update(node*2+2, (begin+end)/2, end, left, right, mark);
    itvTree[node].value = min(itvTree[node*2+1].value, itvTree[node*2+2].value);
}
```

### # 区间树应用
 - 区间最值查询(RMQ, Range Minimum/Maximum Query)
 - 单节点更新或连续区间修改的动态查询
 - 多维空间动态查询
