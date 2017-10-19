---
title: POJ2135 - Farm Tour（最小费用流 + 模板 + SPFA + Dijstra）
date: 2017-05-22 14:58:41
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://poj.org/problem?id=2135


--------------------------
# 题目大意：

 现在有 N 个节点，有M条边，要从 1 走到 N 然后再回到 1 。要求走的边不能重复，求最短路径。

----------------------------------
# 解题过程：

之前看了最小费用最大流然后一直没有做题，于是找了一个模板题来刷，对着板子敲上去居然一次AC，然后又改了下最短路的算法，AC。


-------------------------
# 题目分析：

 算是一个隐含的最小费用最大流，设每条边的容量为1，花费为路径长度。那么所求的就是一个从起点 1 到终点 N 流量为 2 的流的最小费用流。

这里用的是最短增广路算法。

# AC代码：

```cpp
#include <cstdio>
#include <vector>
#include <queue>
#include <cstring>
using namespace std;

const int MAX = 1123, INF = 0x3f3f3f3f;
int N, M;

struct Node{
    int to, cap, cost, rev;
    Node(int to, int cap, int cost, int rev):to(to), cap(cap), cost(cost), rev(rev){}
};

vector<Node> edge[MAX];
int dist[MAX], vis[MAX], prevv[MAX], preve[MAX];

void add_edge(int from, int to, int cap, int cost) {
    edge[from].push_back(Node(to, cap, cost, edge[to].size()));
    edge[to].push_back(Node(from, 0, -cost, edge[from].size()-1));
}

int min_cost_flow(int s, int t, int f) {
    int rst = 0;
    //循环到到达了f流量
    while (f > 0) {
        memset(dist, INF, sizeof(dist));
        queue<int> q;
        q.push(s);
        vis[s] = 1;
        dist[s] = 0;
        while (!q.empty()) {
            int u = q.front();
            for (int i = 0; i < edge[u].size(); i++) {
                Node& e = edge[u][i];
                if (e.cap > 0 && dist[e.to] > dist[u] + e.cost) {
                    dist[e.to] = dist[u] + e.cost;
                    prevv[e.to] = u;
                    preve[e.to] = i;
                    if (!vis[e.to]) {
                        q.push(e.to);
                        vis[e.to] = 1;
                    }
                }
            }
            q.pop();
            vis[u] = 0;
        }
        if (dist[t] == INF) return -1;

        //最路径上最小流量
        int d = f;
        for (int u = t; u != s; u = prevv[u]) {
            d = min(d, edge[prevv[u]][preve[u]].cap);
        }
        //剩余的流量
        f -= d;
        //计算费用
        rst += d * dist[t];
        //修改路上所经过的边的容量
        for (int u = t; u != s; u = prevv[u]) {
            Node& e = edge[prevv[u]][preve[u]];
            e.cap -= d;
            edge[u][e.rev].cap += d;
        }
    }
    return rst;
}

int main() {
    scanf("%d %d", &N, &M);
    for (int i = 0; i < M; i++) {
        int from, to, cost;
        scanf("%d %d %d", &from, &to, &cost);
        add_edge(from, to, 1, cost);
        add_edge(to, from, 1, cost);
    }
    //求流量为2的最小费用
    int ans = min_cost_flow(1, N, 2);
    printf("%d\n", ans);
}
```

#居然可以AC的Dijstra代码：
```
#include <cstdio>
#include <vector>
#include <queue>
#include <cstring>
using namespace std;

const int MAX = 1123, INF = 0x3f3f3f3f;
int N, M;

struct Node{
    int to, cap, cost, rev;
    Node(int to, int cap, int cost, int rev):to(to), cap(cap), cost(cost), rev(rev){}
};

vector<Node> edge[MAX];
int dist[MAX], vis[MAX], prevv[MAX], preve[MAX];

void add_edge(int from, int to, int cap, int cost) {
    edge[from].push_back(Node(to, cap, cost, edge[to].size()));
    edge[to].push_back(Node(from, 0, -cost, edge[from].size()-1));
}

int min_cost_flow(int s, int t, int f) {
    int rst = 0;
    while (f > 0) {
        memset(dist, INF, sizeof(dist));
        queue<int> q;
        q.push(s);
        vis[s] = 1;
        dist[s] = 0;
        while (!q.empty()) {
            int u = q.front();
            for (int i = 0; i < edge[u].size(); i++) {
                Node& e = edge[u][i];
                if (e.cap > 0 && dist[e.to] > dist[u] + e.cost) {
                    dist[e.to] = dist[u] + e.cost;
                    prevv[e.to] = u;
                    preve[e.to] = i;
                    if (!vis[e.to]) {
                        q.push(e.to);
                        vis[e.to] = 1;
                    }
                }
            }
            q.pop();
            vis[u] = 0;
        }
        if (dist[t] == INF) return -1;

        int d = f;
        for (int u = t; u != s; u = prevv[u]) {
            d = min(d, edge[prevv[u]][preve[u]].cap);
        }
        f -= d;
        rst += d * dist[t];
        for (int u = t; u != s; u = prevv[u]) {
            Node& e = edge[prevv[u]][preve[u]];
            e.cap -= d;
            edge[u][e.rev].cap += d;
        }
    }
    return rst;
}

int main() {
    scanf("%d %d", &N, &M);
    for (int i = 0; i < M; i++) {
        int from, to, cost;
        scanf("%d %d %d", &from, &to, &cost);
        add_edge(from, to, 1, cost);
        add_edge(to, from, 1, cost);
    }
    int ans = min_cost_flow(1, N, 2);
    printf("%d\n", ans);
}
```