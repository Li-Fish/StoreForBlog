---
title: UVA4885 - Task（差分约束）
date: 2018-01-22 19:14:16
categories: [ACM, 图论]
tags:
---
# 题目链接：

[题目链接](http://vj.bit-studio.cn/problem/UVALive-4885)

# 题目大意：

给出 N 个任务，M 个约束条件，条件有两种类型。

A 任务要在 B 之后至少 C 分钟后执行。

A 任务要在 B 之后至多 C 分钟内执行。

满足约束条件即可，不用不同任务可以在同一分钟执行。

$1 \le N \le 100$

# 题目分析：

假设 $F(A)$ 为 A 任务执行的时间，那么对于上面两个条件可以化为：

$F(A) - F(B) \ge C$

$F(A) - F(B) \le C$

$F(A) \ge F(B) $

注意这里对第二个约束条件，要转化为两个不等式，如果只有一个等式的话，不能保证 
A 任务的执行在 B 任务之后。

由于题目要求执行的时间不能超过 $10^6$，那么这里对没一个对所有的 i j 加一个条件:

$F(i) - F(j) \le 10^6 - 1$ ，这样同时使得图连通。

最后将上述条件转化为对应的边跑最短路或者最长路即可。


# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 200000;
const int INF = 0x3f3f3f3f;

char op[MAX];

struct {
    int v, w, nxt;
} edge[MAX];

int head[MAX], etot;

void add_edge(int u, int v, int w) {
    edge[etot].v = v;
    edge[etot].w = w;
    edge[etot].nxt = head[u];
    head[u] = etot++;
}

int dist[MAX], num[MAX];
bool vis[MAX];
int n, m;

bool spfa() {
    memset(dist, INF, sizeof(dist));
    memset(num, 0, sizeof(num));
    memset(vis, 0, sizeof(vis));
    queue<int> q;
    q.push(1);
    vis[1] = true;
    dist[1] = 1;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        vis[u] = false;
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            int w = edge[i].w;
            if (dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                if (!vis[v]) {
                    num[v]++;
                    if (num[v] > n + 10) return false;
                    q.push(v);
                    vis[v] = true;
                }
            }
        }
    }
    return true;
}

int main() {
    while (~scanf("%d", &n) && n) {
        scanf("%d", &m);
        etot = 0;
        memset(head, -1, sizeof(head));
        for (int i = 1; i <= m; i++) {
            int u, v, w;
            scanf("%*s %d %*s %s", &u, op);
            if (op[0] == 'a') {
                scanf("%*s %d %*s %*s %*s %*s %d", &w, &v);
                add_edge(u, v, -w);
            } else {
                scanf("%d %*s %*s %*s %*s %*s %*s %*s %d", &w, &v);
                add_edge(v, u, w);
                add_edge(u, v, 0);
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (i == j) continue;
                add_edge(j, i, 1000000 - 2);
            }
        }
        if (!spfa()) puts("Impossible.");
        else {
            int minn = INF;
            for (int i = 1; i <= n; i++) {
                minn = min(minn, dist[i]);
            }
            for (int i = 1; i <= n; i++) {
                printf("%d%c", dist[i] - minn + 1, i == n ? '\n' : ' ');
            }
        }
    }
}
```
# 解题过程：

漏掉了第二个约束条件要转化为两个不等式，ORZ