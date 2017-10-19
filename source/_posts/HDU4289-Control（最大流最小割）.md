---
title: HDU4289 - Control（最大流最小割）
date: 2017-08-08 10:56:44
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=4289

--------------------
# 题目大意：
给出一个无向图，每个点有点权，要删除一些点使得A、B两点不再连通，问最小权值和。

-------------------
# 解题过程：
想到大概是网络流，但是不知道该怎么建图，最后没做出来，看了题解后才发现，原来花费可以和容量交换一下....

--------------------
# 题目分析：
对于每个点拆成入点和出点，中间建一条容量为点权的边。
对原图的无向边(u,v)对u的出点建一条到v的入点边，对v的出点建一条到u的入点的边，容量为无穷大。

这样问题就转化为，要用一些点的入点和出点之间的边去阻塞路径，阻塞所需要的流量就是所需要的花费，要最小花费就是求最小割。

----------------------
# AC代码：
```cpp

#include <bits/stdc++.h>

using namespace std;

const int MAX = 1000;
const int INF = 0x3f3f3f3f;

struct Node {
    int v, nxt, cap;
} edge[MAX * MAX];

int head[MAX], etot;
int level[MAX];

void add_edge(int u, int v, int cap1) {
    edge[etot].v = v;
    edge[etot].cap = cap1;
    edge[etot].nxt = head[u];
    head[u] = etot++;

    edge[etot].v = u;
    edge[etot].cap = 0;
    edge[etot].nxt = head[v];
    head[v] = etot++;
}

bool bfs(int s, int t) {
    memset(level, -1, sizeof(level));
    queue<int> q;
    q.push(s);
    level[s] = 0;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            int cap = edge[i].cap;
            if (cap > 0 && level[v] == -1) {
                level[v] = level[u] + 1;
                q.push(v);
            }
        }
    }
    return level[t] != -1;
}

int dfs(int u, int end, int f) {
    if (u == end) return f;
    int flow = 0, d;
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        int cap = edge[i].cap;
        if (cap > 0 && level[v] > level[u] && (d = dfs(v, end, min(f, cap)))) {
            edge[i].cap -= d;
            edge[i ^ 1].cap += d;
            flow += d;
            f -= d;
            if (!f) break;
        }
    }
    if (flow == 0) level[u] = -1;
    return flow;
}

int max_flow(int s, int t) {
    int flow = 0;
    while (bfs(s, t)) {
        flow += dfs(s, t, INF);
    }
    return flow;
}

int main() {
    int n, m, s, t;
    while (~scanf("%d %d", &n, &m)) {
        memset(head, -1, sizeof(head));
        etot = 0;
        scanf("%d %d", &s, &t);

        for (int i = 1; i <= n; i++) {
            int cost;
            scanf("%d", &cost);
            //对入点和出点建边，费用为点权
            add_edge(i, i + n, cost);
        }

        for (int i = 0; i < m; i++) {
            int u, v;
            scanf("%d %d", &u, &v);
            //对原图的边建两条容量为无穷大的边
            add_edge(u + n, v, INF);
            add_edge(v + n, u, INF);
        }

        //求得的最小割即为答案
        printf("%d\n", max_flow(s, t + n));
    }
}
```