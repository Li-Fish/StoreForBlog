---
title: POJ2195 - Going Home（最小费用流+模板）
date: 2017-07-27 09:52:40
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://poj.org/problem?id=2195

-----------------------
# 题目大意：
给出一张二维的图，每个点距离相邻的点花费为1，图上有相同数量的人和房子，每个房子的容量为1，要使得所有人进入到不同的房子里，最小的花费是多少？

------------------
# 解题过程：
就是裸的最小费用流，建图麻烦点，留下来当模板。
不过这题注意一个坑点是题目描述的数据范围不正确。

------------------------
# AC代码：
```cpp
#include <vector>
#include <cstdio>
#include <algorithm>
#include <queue>
#include <cstring>
#include <cmath>
using namespace std;

const int MAX = 10012;
const int INF = 0x3f3f3f3f;

struct Info {
    int x, y;
    bool flag;
    Info(int x, int y, bool flag): x(x), y(y), flag(flag) {}
};

struct Edge {
    int u, v, w, cap, nxt;
}edge[MAX<<2];

vector<Info> point;
int head[MAX], tot;
int dist[MAX], vis[MAX], pre[MAX], flow[MAX];

int get_dist(int a, int b) {
    return abs(point[a].x - point[b].x) + abs(point[a].y - point[b].y);
}

void add_edge(int u, int v, int w) {
    edge[tot].u = u;
    edge[tot].v = v;
    edge[tot].cap = 1;
    edge[tot].w = w;
    edge[tot].nxt = head[u];
    head[u] = tot++;

    edge[tot].u = v;
    edge[tot].v = u;
    edge[tot].cap = 0;
    edge[tot].w = -w;
    edge[tot].nxt = head[v];
    head[v] = tot++;
}

void spfa(int s) {
    memset(dist, INF, sizeof(dist));
    queue<int> q;
    q.push(s);
    //flow记录当前节点允许的最大流量
    flow[s] = INF;
    vis[s] = 1;
    dist[s] = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            if (edge[i].cap > 0 && dist[v] > dist[u] + edge[i].w) {
                dist[v] = dist[u] + edge[i].w;
                //pre记录当前节点是又那条边过来的
                pre[v] = i;
                flow[v] = min(flow[u], edge[i].cap);
                if (!vis[v]) { q.push(v); vis[v] = 1; }
            }
        }
        vis[u] = 0;
    }
}

int min_cost_flow(int s, int e) {
    int rst = 0;
    while (true) {
        //用spfa求得的最短路增广
        spfa(s);
        //如果和终点无法连通，那么结束
        if (dist[e] == INF) break;
        int d = flow[e], u = e;
        //计算花费
        rst += d * dist[e];
        //更新残量网络
        while (u != s) {
            int last = pre[u];
            edge[last].cap -= d;
            edge[last^1].cap += d;
            u = edge[last].u;
        }
    }
    return rst;
}

int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m) && (n + m)) {
        memset(head, -1, sizeof(head));
        point.clear();
        tot = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                char ch;
                scanf(" %c", &ch);
                if (ch == 'H') point.push_back(Info(i, j, true));
                if (ch == 'm') point.push_back(Info(i, j, false));
            }
        }
        int s = point.size();
        int e = s + 1;
        for (int i = 0; i < point.size(); i++) {
            if (!point[i].flag) {
                add_edge(s, i, 0);
                for (int j = 0; j < point.size(); j++) {
                    if (!point[j].flag) continue;
                    add_edge(i, j, get_dist(i, j));
                }
            }
            else add_edge(i, e, 0);
        }
        printf("%d\n", min_cost_flow(s, e));
    }
}

```