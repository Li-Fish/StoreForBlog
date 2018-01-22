---
title: HDU4280 - Island Transport（最大流+Dinic模板）
date: 2017-07-27 08:42:16
categories:  [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=4280

---------------------------
# 题目大意：
给出$N$个点的坐标和$M$条有容量的边，找出从最西边的点到最东边的点的最大流量。

-------------------------
# 解题过程：
自己的 Dinic 板子一直 TLE，然后抄的学长的，于是以后就打算用这个板子搞 Dinic 了。

-----------------------
# 题目分析：
略

---------------------
# AC代码：
```cpp
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <algorithm>
#include <queue>
using namespace std;
const int INF = 0x3f3f3f3f;
const int MAX = 100000+100;
struct Info {
    int to, cap, nxt;
} edge[MAX<<1];
int head[MAX], tot, cur[MAX];
int N, M, T;
int level[MAX];
//用前向星建边
void add_edge(int u, int v, int w) {
    edge[tot].to = v;
    edge[tot].cap = w;
    edge[tot].nxt = head[u];
    head[u] = tot++;
    edge[tot].to = u;
    edge[tot].cap = w;
    edge[tot].nxt = head[v];
    head[v] = tot++;
}
//BFS构建层次图
bool bfs(int s, int e) {
    memset(level, -1, sizeof(level));
    queue<int> q;
    level[s] = 0;
    q.push(s);
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].to;
            if (edge[i].cap > 0 && level[v] == -1) {
                level[v] = level[u] + 1;
                q.push(v);
            }
        }
    }
    return level[e] != -1;
}
int dfs(int u, int end, int f) {
    //如果为终点的话就返回流量
    if (u == end) return f;
    int flow = 0, d;
    for (int &i = cur[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].to;
        //只由低层次的点向高层次的点增广
        if (edge[i].cap > 0 && level[u] < level[v] && (d = dfs(v, end, min(edge[i].cap, f)))) {
            edge[i].cap -= d;
            edge[i^1].cap += d;
            flow += d;
            f -= d;
            if (!f) break;
        }
    }
    //如果当前点的流量为0，那么排除掉这个点
    if (flow == 0) level[u] = -1;
    return flow;
}
int max_flow(int s, int e) {
    int flow = 0;
    while (bfs(s, e)) {
        memcpy(cur, head, sizeof(head));
        flow += dfs(s, e, INF);
    }
    return flow;
}
int main() {
    scanf("%d", &T);
    while (T--) {
        tot = 0;
        scanf("%d %d", &N, &M);
        int sx = INF, ex = -INF, s, e;
        for (int i = 1; i <= N; i++) {
            head[i] = -1;
            int x, y;
            scanf("%d %d", &x, &y);
            if (x < sx) sx = x, s = i;
            if (x > ex) ex = x, e = i;
        }
        for (int i = 1; i <= M; i++) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            add_edge(u, v, w);
        }
        printf("%d\n", max_flow(s, e));
    }
}
```