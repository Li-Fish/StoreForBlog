---
title: POJ3694 - Network（强连通+LCA）
date: 2017-08-11 11:05:45
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：

http://poj.org/problem?id=3694



--------------------
# 题目大意：

给出一个无向图。

有 q 次操作，每次增加一条边，询问图中有多少桥。

q < 1e3, n < 1e5, m < 2e5



-------------------
# 解题过程：

不会写，思路到是有，不知道该怎么维护一条路径上的点，并让其缩点成一个点，到时想起来了树链剖分（昨天刚写了一个模板）。

想法是缩成树型图，每个边权值为1，增加一条边就让路径上的所有边的边权置零，每次输出整棵树的权值和即可。

感觉太麻烦了，于是偷偷去看了下讨论区，发现用 LCA（最近公共祖先） 加并查集就很容易解决这个问题



--------------------
# 题目分析：

首先缩点成树形图，对于每一次操作，新加的一条边 (u, v) ，让 u，v 两点确定的路径上的所有点的用并查集缩成一个点（包括u，v），缩成的点为u，v的LCA，这样缩掉几个点，原图就减少了几个桥。用一个值来维护答案即可。





----------------------
# AC代码：
```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <stack>
#include <algorithm>

using namespace std;

const int MAX = 100000 + 10;

int n, m, q, Case, sum;
int head[MAX], etot;
int deep[MAX], f[MAX], fa[MAX];
int pre[MAX], low[MAX], mark[MAX], dfs_clock, scc_cnt;
stack<int> S;

struct Edge {
    int u, v, nxt;
} edge[MAX << 4];

void add_edge(int u, int v) {
    edge[etot].v = v;
    edge[etot].u = u;
    edge[etot].nxt = head[u];
    head[u] = etot++;
}

void dfs(int u, int fat) {
    pre[u] = low[u] = ++dfs_clock;
    S.push(u);
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (!pre[v]) {
            dfs(v, i);
            low[u] = min(low[u], low[v]);
        } else if (i != (fat^1)) {
            low[u] = min(low[u], pre[v]);
        }
    }
    if (low[u] == pre[u]) {
        int x;
        scc_cnt++;
        do {
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
        } while (x != u);
    }
}

void init() {
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    memset(head, -1, sizeof(head));
    scc_cnt = dfs_clock = sum = etot = 0;
    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        add_edge(u, v);
        add_edge(v, u);
    }
}

void tarjan() {
    for (int i = 1; i <= n; i++) {
        if (!pre[i]) dfs(i, -1);
    }
}

void get_deep(int u, int fat, int dep) {
    deep[u] = dep, f[u] = u;
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (v != fat) {
            fa[v] = u;
            get_deep(v, u, dep + 1);
            sum++;
        }
    }
}

void rebuild() {
    int tail = etot;
    memset(head, -1, sizeof(head));
    for (int i = 0; i < tail; i++) {
        int u = edge[i].u;
        int v = edge[i].v;
        if (mark[u] != mark[v]) {
            add_edge(mark[u], mark[v]);
        }
    }
}

int root(int t) {
    if (t == f[t]) return t;
    return f[t] = root(f[t]);
}

void lca(int u, int v) {
    u = root(u), v = root(v);
    stack<int> st;
    //循环直到找到公共祖先
    while (u != v) {
        //让u为较深的节点
        if (deep[u] < deep[v]) swap(u, v);
        //通过减少的节点数维护答案
        sum--;
        st.push(u);
        //让u爬到他的父亲节点
        u = root(fa[u]);
    }

    //最后让路径上经过的点都缩成LCA一个点
    while (!st.empty()) {
        f[st.top()] = u;
        st.pop();
    }
}

void solve() {
    //边双连通划分+缩点
    tarjan();
    rebuild();

    //获得新图的每个点的深度和父亲
    get_deep(1, -1, 1);

    printf("Case %d:\n", ++Case);

    scanf("%d", &q);
    while (q--) {
        int u, v;
        scanf("%d %d", &u, &v);
        //对每次操作通过并查集缩点
        lca(mark[u], mark[v]);
        printf("%d\n", sum);
    }
}

int main() {
    while (~scanf("%d %d", &n, &m) && (n + m)) {
        init();
        solve();
    }
}
```