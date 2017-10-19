---
title: 社交网络图中结点的“重要性“计算（Dijkstra + SPFA + Floyd + 模板）
date: 2017-03-20 15:56:27
categories: [ACM, 图论, 最短路]
tags:
---
# 题目链接：
 无
 ---------------
# 题目大意：
  求一个点到其他所有点的最短距离和，保证图连通。
  -------------------
# 解题过程：
 刚开始用 Floyd 水过的，后来用换了几种方法，不错的模板题，Floyd 的时候，要用 vector 存边，否则超内存。   

------------------------
# 题目分析
 略

---------------------
# AC代码（Dijkstra + SPFA）
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 11234, INF = 0x3f3f3f3f;

vector<int> edges[MAX];
int dist[MAX], book[MAX];

void spfa(int s) {
    memset(dist, INF, sizeof(dist));
    memset(book, 0, sizeof(book));

    queue<int> q;
    q.push(s);
    book[s] = 1;
    dist[s] = 0;

    while (!q.empty()) {
        int u = q.front();
        for (int i = 0; i < edges[u].size(); i++) {
            int v = edges[u][i];
            if (dist[v] > dist[u] + 1) {
                dist[v] = dist[u] + 1;
                if (!book[v]) {
                    q.push(v);
                    book[v] = 1;
                }
            }
        }
        q.pop();
        book[u] = 0;
    }
}

void dijkstra(int s) {
    memset(dist, INF, sizeof(dist));
    priority_queue<pair<int, int> > q;
    dist[s] = 0;
    q.push(make_pair(-dist[s], s));
    while (!q.empty()) {
        int u = q.top().second;
        q.pop();

        for (int i = 0; i < edges[u].size(); i++) {
            int v = edges[u][i];
            if (dist[v] > dist[u] + 1) {
                dist[v] = dist[u] + 1;
                q.push(make_pair(-dist[v], v));
            }
        }
    }
}

int main() {
    int n, m;
    scanf("%d %d", &n, &m);
    while (m--) {
        int u, v;
        scanf("%d %d", &u, &v);
        edges[u].push_back(v);
        edges[v].push_back(u);
    }
    int k;
    scanf("%d", &k);
    while (k--) {
        int s;
        scanf("%d", &s);
        dijkstra(s);
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            if (i == s)
                continue;
            sum += dist[i];
        }
        printf("Cc(%d)=%.2f\n", s, (n-1.0)/sum);
    }
}

```
------------------------
# AC代码（Floyd）：
```cpp
#include<bits/stdc++.h>
using namespace std;
const int INF = 0x3f3f3f3f, MAX = 10001;

int main()
{
    vector<int>edge[MAX];
    int n, m;
    scanf("%d %d", &n, &m);

    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= n; j++) {
            edge[i].push_back(INF);
        }
    }

    for (int i = 1; i <= n; i++) {
        edge[i][i] = 0;
    }

    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        edge[u][v] = edge[v][u] = 1;
    }

    for (int k = 1; k <= n; k++) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (edge[i][j] > edge[i][k] + edge[k][j])
                    edge[i][j] = edge[i][k] + edge[k][j];
            }
        }
    }

    int k;
    scanf("%d", &k);
    while (k--) {
        int c;
        scanf("%d", &c);
        double sum = 0;
        for (int i = 1; i <= n; i++) {
            if (i == c)
                continue;
            sum += edge[c][i];
        }
        printf("Cc(%d)=%.2f\n", c, (n-1)/sum);
    }
}

```