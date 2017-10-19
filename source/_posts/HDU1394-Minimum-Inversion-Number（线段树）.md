---
title: HDU1394 - Minimum Inversion Number（线段树）
date: 2017-06-21 14:49:36
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=1394

------------------------
# 题目大意：
给定一个由$0$到$n-1$组成，长度为$n$每个元素唯一的序列，可以进行一种操作，把第一个元素放到最后一个位置。求经过若干次操作后的，最小逆序对数。


----------------------------
# 解题过程：
这题之前写过一个暴力解法的题解，现在用线段树来解决一下。

-------------------------
# 题目分析：
这里用线段树主要是求解初始状态的逆序对数，对于每次的操作有一个结论可以用。

要求逆序对数，那么对于每个数我要求在这个数之前有多少个大于这个数的元素。

因为序列的元素是从$0$到$n-1$的，那么我用$n$个叶子节点去维护这些值是否出现过，出现置$1$否则为$0$，对于非叶子节点就维护区间内数字出现的个数，那么我要查询比$a$大的数有几个，那么我只需要查询$[a, n]$这个区间的值就好了。

这题比较容易做，其他题可能也会用到这个思想，不过数字不是从$0$到$n-1$的，需要离散化一下。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

#define lson root<<1
#define rson root<<1|1
#define MID int m = (l + r) / 2

const int MAX = 5000+10;

int data[MAX], tree[MAX<<2];

void build(int root, int l, int r) {
    if (l == r) {
        tree[root] = 0;
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m+1, r);
    tree[root] = tree[lson] + tree[rson];
}

void updata(int root, int l, int r, int pos) {
    if (pos < l || pos > r)
        return;
    if (l == r) {
        tree[root] = 1;
        return;
    }
    MID;
    updata(lson, l, m, pos);
    updata(rson, m+1, r, pos);
    tree[root] = tree[lson] + tree[rson];
}

int query(int root, int l, int r, int ql, int qr) {
    if (qr < l || r < ql)
        return 0;
    if (ql <= l && r <= qr) {
        return tree[root];
    }
    MID;
    return query(lson, l, m, ql, qr) + query(rson, m+1, r, ql, qr);
}

int main() {
    int n;
    while (~scanf("%d", &n)) {
        build(1, 1, n);
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            scanf("%d", &data[i]);
            data[i] += 1;
            //这里处理了一下，下标从1开始比较方便
            //大于data[i]，查询[data[i], n]区间的值就是i之前比data[i]大的元素的个数
            sum += query(1, 1, n, data[i], n);
            updata(1, 1, n, data[i]);
        }
        int rst = sum;
        for (int i = 1; i <= n; i++) {
            sum += n - data[i];
            sum -= data[i] - 1;
            rst = min(sum, rst);
        }
        printf("%d\n", rst);
    }
}
```