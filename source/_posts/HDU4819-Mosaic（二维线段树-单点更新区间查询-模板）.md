---
title: HDU4819 - Mosaic（二维线段树-单点更新区间查询 + 模板）
date: 2017-10-27 18:11:19
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：

https://vjudge.net/problem/HDU-4819



# 题目大意：

给出一个 n * n 的矩阵，每个位置有一个整数值。进行 q 次操作，每次选矩阵的一个元素为中心，取以这个元素为中心的 L * L 的最大值和最小值，将这个元素的值赋值成最大值最小值的平均值。

$n \le 800, q \le 100000$



# 题目分析：

裸的二维线段树，单点修改，询问区间最值。

其实二维的线段树就是一个行线段树套列线段树，注意进行更新的时候，不能直接赋值修改，只修改行线段树叶子节点里面列线段树的叶子节点，然后向上合并。

# AC代码：

```cpp
#include <bits/stdc++.h>

using namespace std;

const int INF = 0x3f3f3f3f;
const int MAX = 1010;

#define lson o<<1
#define rson o<<1|1
#define MID int m = (l + r) >> 1


//列线段树，用来维护列的节点
struct Nodey {
    int Max, Min;

    Nodey operator+(const struct Nodey &t) {
        Nodey rst;
        rst.Max = max(Max, t.Max);
        rst.Min = min(Min, t.Min);
        return rst;
    }
};

int locy[MAX], locx[MAX];


//行线段树，用来维护行的节点
struct Nodex {
    Nodey sty[MAX << 2];

    void build(int o, int l, int r) {
        sty[o].Max = -INF;
        sty[o].Min = INF;
        if (l == r) {
            locy[l] = o;
            return;
        }
        MID;
        build(lson, l, m);
        build(rson, m + 1, r);
    }

    Nodey query(int o, int l, int r, int ql, int qr) {
        if (qr < l || r < ql) return (Nodey) {-INF, INF};
        if (ql <= l && r <= qr) return sty[o];
        MID;
        return query(lson, l, m, ql, qr) + query(rson, m + 1, r, ql, qr);
    }
} stx[MAX << 2];

int n;

void build(int o, int l, int r) {
    stx[o].build(1, 1, n);
    if (l == r) {
        locx[l] = o;
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m + 1, r);
}


//进行单点更新，这里首先更新了叶子节点，然后向上合并父亲节点；
void Modify(int x, int y, int val) {
    int tx = locx[x];
    int ty = locx[y];
    stx[tx].sty[ty].Min = stx[tx].sty[ty].Max = val;
    for (int i = tx; i; i >>= 1) {
        for (int j = ty; j; j >>= 1) {
            if (i == tx && j == ty) continue;
            if (j == ty) {
                //如果当前更新的列就是需要更新的叶子节点，那么由当前行的两个儿子节点来更新信息
                stx[i].sty[j] = stx[i << 1].sty[j] + stx[i << 1 | 1].sty[j];
            } else {
                //否则由当前列的如果儿子节点来更新
                stx[i].sty[j] = stx[i].sty[j << 1] +  stx[i].sty[j << 1 | 1];
            }
        }
    }
}

Nodey query(int o, int l, int r, int ql, int qr, int y1, int y2) {
    if (qr < l || r < ql) return (Nodey) {-INF, INF};
    if (ql <= l && r <= qr) return stx[o].query(1, 1, n, y1, y2);
    MID;
    return query(lson, l, m, ql, qr, y1, y2) + query(rson, m + 1, r, ql, qr, y1, y2);
}


int main() {
    int T;
    scanf("%d", &T);
    int Case = 0;
    while (T--) {
        Case++;
        printf("Case #%d:\n", Case);
        scanf("%d", &n);
        build(1, 1, n);
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                int a;
                scanf("%d", &a);
                Modify(i, j, a);
            }
        }
        int q;
        int x, y, L;
        scanf("%d", &q);
        while (q--) {
            scanf("%d %d %d", &x, &y, &L);
            int x1 = max(x - L / 2, 1);
            int x2 = min(x + L / 2, n);
            int y1 = max(y - L / 2, 1);
            int y2 = min(y + L / 2, n);
            Nodey ans = query(1, 1, n, x1, x2, y1, y2);
            int t = (ans.Max + ans.Min) / 2;
            printf("%d\n", t);
            Modify(x, y, t);
        }
    }
}
```
# 解题过程：

晚上模拟赛的题，感觉是一个二维线段树的裸题，但是不会，马上要去CCPC秦皇岛了，现学的。