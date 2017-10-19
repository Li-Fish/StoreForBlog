---
title: POJ 3468 - A Simple Problem with Integers（线段树区间更新+模板）
date: 2017-02-10 17:38:30
categories: [ACM,  数据结构, 线段树]
tags:
---
# 题目链接：
------------------
http://poj.org/problem?id=3468
# 题目描述：
------------
给n个整数，进行m次查询或更新，查询指区间[l ,r]整数的和，更新指区间[l ,r]的整数全部增加z。
# 解题过程
-----------------
题目不难，妥妥的模板题，卡在 pushDown 函数上两次，还不够细心。

首先是lazy标记没处理好，这个提应该是让 **lazy += z**， 我第一次写成了 **lazy = z**，导致标记直接被替换掉了，之前的更新没加上。

第二次卡在了long long上面， 全部检查了一遍， 感觉应该都用 long long 了， 最后又仔细看了一遍， 发现 pushDown 函数里面的 lazy 是和用来标记左右区间的 l, r 一起定义的，改了之后就AC了。

讲道理，这个lazy还真是有点懒，更新的时候，能不更新下一步就坚决不更新下一步，然后所有事都拖到必须要干的时候才干23333。
# 题目分析：
------
+ 首先由于线段树是完全二叉树，建树使用数组就可以。
+ 要写一个pushDown函数， 用来把lazy标记传给他的两个儿子。
+ query 和 upDate 函数，每次如果向下递归的时候，都要首先检查下当前节点是不是有lazy标记，如果有的话就传给下一级。
+ 最后有点要注意的是，本题的更新是数字的值增加，不是直接替换，同理添加或传递lazy标记的时候也要以 += 的形式， 不能直接替换掉原来的。

# AC代码：
```cpp
#include<cstdio>
using namespace std;

struct node
{
    long long lazy;
    long long value;
    int l, r;
};

node store[100000*100];
long long data[100000*100];

void build(int l, int r, int root)
{
    store[root].l = l, store[root].r = r;
    store[root].lazy = 0;

    if (l == r)
    {
        store[root].value = data[l];
        return;
    }

    int m = (l + r) / 2;

    build(l, m, root*2+1);
    build(m+1, r, root*2+2);

    store[root].value = store[root*2+1].value + store[root*2+2].value;
    return;
}

void pushDown(int root)
{
    if (store[root].lazy != 0)
    {
        long long l, r;
        long long z;
        z = store[root].lazy;

        store[root*2+1].lazy += z;
        l = store[root*2+1].l, r = store[root*2+1].r;
        store[root*2+1].value += (r - l + 1) * z;

        store[root*2+2].lazy += z;
        l = store[root*2+2].l, r = store[root*2+2].r;
        store[root*2+2].value += (r - l + 1) * z;

        store[root].lazy = 0;
    }
    return;
}

void upDate(int l, int r, int root, long long z)
{
    int nl = store[root].l, nr = store[root].r;

    if (l <= nl && r >= nr)
    {
        store[root].lazy += z;
        store[root].value += (nr - nl + 1) * z;
        return;
    }

    pushDown(root);

    int m = (nl + nr) / 2;

    if (l <= m)
        upDate(l, r, root*2+1, z);
    if (r > m)
        upDate(l, r, root*2+2, z);

    store[root].value = store[root*2+1].value + store[root*2+2].value;
    return;
}

long long query(int l, int r, int root)
{
    int nl = store[root].l, nr = store[root].r;

    if (l <= nl && r >= nr)
        return store[root].value;

    pushDown(root);
    int m = (nl + nr) / 2;

    long long sum = 0;
    if (l <= m)
        sum += query(l, r, root*2+1);
    if (r > m)
        sum += query(l, r, root*2+2);

    return sum;
}

int main()
{
//    freopen("in", "r", stdin);
    int n, q;
    scanf("%d %d", &n, &q);
    for (int i = 0; i < n; i++)
        scanf("%lld", data+i);

    build(0, n-1, 0);

    char mode[11];
    int l, r;
    long long z;
    while (q--)
    {
        scanf("%s %d %d", mode, &l, &r);
        if (mode[0] == 'Q')
            printf("%lld\n", query(l-1, r-1, 0));
        else if (mode[0] == 'C')
        {
            scanf("%lld", &z);
            upDate(l-1, r-1, 0, z);
        }
    }
}





```