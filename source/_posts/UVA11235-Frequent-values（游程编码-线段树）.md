---
title: UVA11235 - Frequent values（游程编码+线段树）
date: 2017-06-28 16:56:32
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
https://vjudge.net/problem/UVA-11235


-------------------
# 题目大意：
给定一个递增序列，询问一段区间内出现频率最多的数出现的次数。

----------------------
# 解题过程：
之前图灵杯比赛的题，当时照着板子敲A的，现在突发奇想补一下题，感觉还是挺简单的，就是用到了游程编码。

-----------------------------
# 题目分析：
由于题目是有序的，那么可以很方便的将一段相同的数合为一个区间，`data[i]`表示第i段区间元素个数，`lb[i]`记录第`i`段区间的左边界，`rb[i]`记录第`i`段区间的右边界，用`id[i]`表示原序列的第i个数处于哪个区间。

那么对于查询l到r的区间，如果`l`和`r`在同一段区间内，那么显然答案就是`r-l+1`。
如果l和r不在同一段区间，那么答案为以下三个中的最大值，`rb[id[l]] - l + 1`,` r - lb[id[r]] + 1`, `id[l]+1`区间到`id[r]-1`区间中的最大值`。


-------------------
# AC代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 112345;

int tree[MAX<<2];
int data[MAX];

#define lson o<<1
#define rson o<<1|1
#define MID int m = (l+r)/2

void build(int o, int l, int r) {
    if (l == r) {
        tree[o] = data[l];
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m+1, r);
    tree[o] = max(tree[lson], tree[rson]);
}

int query(int o, int l, int r, int ql, int qr) {
    if (qr < l || r < ql) return 0;
    if (ql <= l && r <= qr) {
        return tree[o];
    }
    MID;
    return max(query(lson, l, m, ql, qr), query(rson, m+1, r, ql, qr));
}

int raw_data[MAX], n, q, lb[MAX], rb[MAX], id[MAX], top;

void init() {
    top = 1;
    lb[top] = data[top] = id[1] = 1;
    for (int i = 2; i <= n+1; i++) {
        if (raw_data[i] != raw_data[i-1] || i == n+1) {
            rb[top++] = i-1;
            lb[top] = i;
        }
        data[top]++;
        id[i] = top;
    }
}

int main() {
    //freopen("in", "r", stdin);
    //freopen("out", "w", stdout);
    while (~scanf("%d", &n) && n) {
        scanf("%d", &q);
        for (int i = 1; i <= n; i++) {
            scanf("%d", raw_data+i);
        }
        memset(data, 0, sizeof(data));
        init();
        n = top-1;
        build(1, 1, n);
        int l, r;
        while (q--) {
            scanf("%d %d", &l, &r);
            //如果l和r在同一个区间内
            if (id[l] == id[r]) {
                printf("%d\n", r-l+1);
                continue;
            }
            //l和r不在一个区间内
            int a = id[l], b = id[r];
            a = rb[a] - l + 1;
            b = r - lb[b] + 1;
            int t = max(a, b);
            a = id[l] + 1, b = id[r] - 1;
            if (a <= b) {
                //查询id[l]+1区间到id[r]-1区间中的最大值
                t = max(t, query(1, 1, n, a, b));
            }
            printf("%d\n", t);
        }
    }
}
```