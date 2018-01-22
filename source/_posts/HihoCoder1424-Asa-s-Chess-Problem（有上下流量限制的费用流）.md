---
title: HihoCoder1424 - Asa's Chess Problem（有上下流量限制的费用流）
date: 2017-10-16 07:53:42
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：

[https://vjudge.net/problem/HihoCoder-1424](https://vjudge.net/problem/HihoCoder-1424)

# 题目大意：

参考 [http://www.cnblogs.com/flipped/p/7635420.html](http://www.cnblogs.com/flipped/p/7635420.html)

> 有个 N×N 的棋盘，告诉你每个格子黑色(1)或白色(0)，以及每对能相互交换的同行或同列格子，每个格子只在一对中，即共有N×N/2对。求最少交换次数使得每行每列的黑格子总数满足给出的上下范围：若最终第i行,第j列分别有R[i],C[j]个黑格子，那么需要让Rl[i]≤R[i]≤Rh[i],Cl[j]≤C[i]≤Ch[j]。



# 题目分析：

这里先介绍一种有流量下限限制的建图方式，参考[这个博客](http://www.cnblogs.com/kane0526/archive/2013/04/05/3001108.html)。

记节点 i 所有流入的流量下限和为 in[i]，所有的流出流入和下限为 out[i]，建一个超级源点 SS，超级汇点 ST。

如果一个节点 in[i] > out[i]，那么建一条 SS 到 i 的边，流量为 in[i] - out[i]。

如果 in[i] < out[i]，那么建一条 i 到 ST 的边，流量为 out[i] - in[i]。

对于无源汇的图来说，上面从 SS 到 ST跑一个最大流，如果上面的从 SS 出发的附加边满流，当前就是一个可行流，否则无解。

对于有源汇的图来说，需要从 T 到 S 连一条流量为无穷的边，然后再从 SS 到 ST 跑最大流。 



对于这个题，设每一行每一列原有的黑色棋子数量为 R[i] 和 C[i]。

+ 首先从 S 到每一行每一列建一条上下限均为 R[i] 或 C[i] 的边
+ 每一行每一列对 T 建边，容量上下限为 Rl[i]， Rh[i] 或 Cl[i]，Ch[i]
+ 然后对于可以交换的棋子，如果他们颜色相同，那么不需要建边，否则如果列相同，黑色所在的行向白色棋子所在的行建流量下限为 0 上限为 1 费用为 1 的边，列相同类似
+ 从 t 到 s 建一条流量下限为 0，上限为无穷的边。

上述所有边默认费用为 0 。

# AC代码：

```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 600;

int n;
int data[112][112];

struct Edge {
    int u, v, c, w, nxt;
} edge[MAX * MAX];

int head[MAX], etot;

int in[112], out[112];

void add_edge(int u, int v, int low, int up, int w) {
    edge[etot].u = u;
    edge[etot].v = v;
    edge[etot].c = up - low;
    edge[etot].w = w;
    edge[etot].nxt = head[u];
    head[u] = etot++;

    out[u] += low;

    edge[etot].u = v;
    edge[etot].v = u;
    edge[etot].c = 0;
    edge[etot].w = -w;
    edge[etot].nxt = head[v];
    head[v] = etot++;

    in[v] += low;
}

int dist[MAX], vis[MAX], pre[MAX], flow[MAX];

void spfa(int s) {
    memset(dist, 0x3f, sizeof(dist));
    queue<int> q;
    q.push(s);
    flow[s] = 1e9;
    vis[s] = 1;
    dist[s] = 0;

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            Edge &e = edge[i];
            if (edge[i].c > 0 && dist[e.v] > dist[e.u] + e.w) {
                dist[e.v] = dist[e.u] + e.w;
                pre[e.v] = i;
                flow[e.v] = min(flow[e.u], e.c);
                if (!vis[e.v]) {
                    vis[e.v] = true;
                    q.push(e.v);
                }
            }
        }
        vis[u] = false;
    }
}

pair<int, int> min_cost_flow(int s, int e) {
    int rst = 0, total = 0;
    while (true) {
        spfa(s);
        if (dist[e] == 0x3f3f3f3f) break;
        int d = flow[e], u = e;
        total += d;
        rst += dist[e] * d;
        while (u != s) {
            int last = pre[u];
            edge[last].c -= d;
            edge[last ^ 1].c += d;
            u = edge[last].u;
        }
    }
    return make_pair(total, rst);
}


int Rl[112], Rh[112], Cl[112], Ch[112];

int row[112], col[112];

int main() {
    while (~scanf("%d", &n)) {

        etot = 0;
        memset(head, -1, sizeof(head));
        memset(col, 0, sizeof(col));
        memset(row, 0, sizeof(row));
        memset(in, 0, sizeof(in));
        memset(out, 0, sizeof(out));


        //源点，汇点，超级源点，超级汇点
        int s = 0;
        int e = n * 2 + 1;
        int ss = e + 1;
        int se = ss + 1;

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                scanf("%d", &data[i][j]);
                row[i] += data[i][j];
                col[j] += data[i][j];
            }
        }

        for (int i = 1; i <= n; i++) {
            scanf("%d %d", &Rl[i], &Rh[i]);
        }
        for (int i = 1; i <= n; i++) {
            scanf("%d %d", &Cl[i], &Ch[i]);
        }

        for (int i = 1; i <= n; i++) {
            add_edge(s, i, row[i], row[i], 0);
            add_edge(s, i + n, col[i], col[i], 0);
            add_edge(i, e, Rl[i], Rh[i], 0);
            add_edge(i + n, e, Cl[i], Ch[i], 0);
        }

        int sum = 0;

        for (int i = 1; i <= n * n / 2; i++) {
            int x1, y1, x2, y2;
            scanf("%d %d %d %d", &x1, &y1, &x2, &y2);
            if (data[x1][y1] == data[x2][y2]) continue;
            if (data[x1][y1] == 0) swap(x1, x2), swap(y1, y2);
            if (x1 == x2) {
                add_edge(n + y1, n + y2, 0, 1, 1);
            } else if (y1 == y2) {
                add_edge(x1, x2, 0, 1, 1);
            }
        }

        add_edge(e, s, 0, 1e9, 0);

        //对超级源点，超级汇点建边
        for (int i = 0; i <= n + n + 1; i++) {
            int t = in[i] - out[i];
            if (t < 0) {
                t = -t;
                add_edge(i, se, 0, t, 0);
            } else {
                sum += t;
                add_edge(ss, i, 0, t, 0);
            }
        }


        pair<int, int> ans = min_cost_flow(ss, se);

        if (ans.first != sum) {
            printf("-1\n");
        } else {
            printf("%d\n", ans.second);
        }
    }
}
```
# 解题过程：

这个题感觉也不是很难，感觉应该做出来的，关键是比赛的时候漏看了一个条件，只有列或行相同时才可以交换，如果没有这个条件建图就复杂了，当时也想麻烦了。