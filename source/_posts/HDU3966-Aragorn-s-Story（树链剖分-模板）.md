---
title: HDU3966 - Aragorn's Story（树链剖分+模板）
date: 2017-08-09 19:17:36
categories: [ACM, 数据结构, 树链剖分]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=3966


--------------------
# 题目大意：
给出一颗树，每个点有点权。
两种操作：
1. 询问一个点的权值。
2. 令一条路径上的所有点权值加value

点数5e4

-------------------
# 解题过程：
线段树的模板题，拿着学长的板子直接上了，结果又RE了，没改MAX大小...
然后又交了一发TLE，用来存边的vector没初始化...
然后又交了一发WA，push_down的时候直接把子节点的lazy标记覆盖了（这个地方不知道错了多少次了！！！）

--------------------
# 题目分析：
树链剖分的模板题

总的来说树链剖分就是一个工具，可以把一个路径上的点映射成几段连续的区间上。这样对于连续的区间可以用线段树维护，对于DFS序也是这样。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

#define lson o<<1
#define rson o<<1|1
#define MID int m = (l+r)/2

using namespace std;

const int MAX = 50000 + 10;
const int INF = 0x3f3f3f3f;

struct Info {
    int sum, lazy, cnt;

    Info() {
        sum = lazy = cnt = 0;
    }

    Info operator+(const Info &a) {
        Info rst;
        rst.sum = sum + a.sum;
        rst.cnt = cnt + a.cnt;
        return rst;
    }
} tree[MAX << 2];

vector<int> edge[MAX];
int data[MAX], n, m, p;
char str[112];

int id_data[MAX];
int fa[MAX], son[MAX], siz[MAX];
int deep[MAX], top[MAX], tid[MAX];
int _cnt;

void build(int o, int l, int r) {
    tree[o].lazy = 0;
    if (l == r) {
        tree[o].sum = id_data[l];
        tree[o].cnt = 1;
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m + 1, r);
    tree[o] = tree[lson] + tree[rson];
}

void push_down(int o) {
    if (!tree[o].lazy) return;
    int lazy = tree[o].lazy;
    tree[lson].lazy += lazy;
    tree[rson].lazy += lazy;
    tree[lson].sum += lazy * tree[lson].cnt;
    tree[rson].sum += lazy * tree[rson].cnt;
    tree[o].lazy = 0;
}

void updata(int o, int l, int r, int ul, int ur, int d) {
    if (r < ul || ur < l) return;
    if (ul <= l && r <= ur) {
        tree[o].sum += tree[o].cnt * d;
        tree[o].lazy += d;
        return;
    }
    push_down(o);
    MID;
    updata(lson, l, m, ul, ur, d);
    updata(rson, m + 1, r, ul, ur, d);
    tree[o] = tree[lson] + tree[rson];
}

Info query(int o, int l, int r, int pos) {
    if (r < pos || pos < l) return Info();
    if (l == r) {
        return tree[o];
    }
    push_down(o);
    MID;
    return query(lson, l, m, pos) + query(rson, m + 1, r, pos);
}

//第一次dfs记录信息
void dffs(int u, int f, int d) {
    siz[u] = 1, deep[u] = d;
    fa[u] = f, son[u] = -1;
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i];
        if (v != f) {
            dffs(v, u, d + 1);
            siz[u] += siz[v];
            if (son[u] == -1 || siz[son[u]] < siz[v]) {
                son[u] = v;
            }
        }
    }
}

//第二次dfs找出重链，进行剖分
void dfss(int u, int t) {
    top[u] = t, tid[u] = _cnt++;
    id_data[_cnt - 1] = data[u];
    if (son[u] != -1) dfss(son[u], t);
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i];
        if (son[u] != v && fa[u] != v) dfss(v, v);
    }
}

void splite() {
    _cnt = 1;
    dffs(1, -1, 0);
    dfss(1, 1);
}

void Updata(int x, int y, int d) {
    int tx = top[x], ty = top[y];
    while (tx != ty) {
        if (deep[tx] < deep[ty]) swap(x, y), swap(tx, ty);
        updata(1, 1, n, tid[tx], tid[x], d);
        x = fa[tx], tx = top[x];
    }
    if (deep[x] < deep[y]) swap(x, y);
    updata(1, 1, n, tid[y], tid[x], d);
}

//Info Query(int x, int y) {
//    Info ix, iy;
//    int tx = top[x], ty = top[y];
//    while (tx != ty) {
//        if (deep[tx] < deep[ty]) swap(x, y), swap(tx, ty), swap(ix, iy);
//        ix = query(1, 1, n, tid[tx], tid[x]) + ix;
//        x = fa[tx], tx = top[x];
//    }
//    if (deep[x] < deep[y]) swap(x, y);
//    return ix + query(1, 1, n, tid[y], tid[x]) + iy;
//}

void init() {
    for (int i = 1; i <= n; i++) {
        scanf("%d", &data[i]);
        edge[i].clear();
    }
    for (int i = 1; i < n; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        edge[u].push_back(v);
        edge[v].push_back(u);
    }
}


void solve() {
    splite();
    build(1, 1, n);
    while (p--) {
        int a, b, c;
        scanf("%s", str);
        if (str[0] == 'Q') {
            scanf("%d", &a);
            printf("%d\n", query(1, 1, n, tid[a]).sum);
        } else {
            scanf("%d %d %d", &a, &b, &c);
            if (str[0] == 'I') {
                Updata(a, b, c);
            } else {
                Updata(a, b, -c);
            }
        }
    }
}

int main() {
    while (~scanf("%d %d %d", &n, &m, &p)) {
        init();
        solve();
    }
}
```