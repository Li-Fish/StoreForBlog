---
title: POJ3259 - Wormholes（连通图判断负环）
date: 2017-06-13 11:10:29
categories: [ACM, 图论, 最短路]
tags:
---
# 题目链接；
http://poj.org/problem?id=3259

------------------------
# 题目大意：
给出N个图，每个图有两种边，一个是无向的正权边，一种是有向的负权边，保证所给的图为连通图，求是否存在负环。

-----------------------------
# 解题过程：
刚开始以为给出的图不连通，然后用Floyd超时，后来问了学长，翻了下POJ的讨论，发现大家都是默认为图连通做的……

然后敲了下Bellman和SPFA判断负环就A了。


--------------------------------
# 题目分析：
因为保证图联通，那么可以假设从任意一点出发。

Bellman：如果松弛操进行N次依然可以松弛，那么存在负环。
SPFA：如果一个点入队次数大于等于N次，那么处在负环。


----------------------------------
# AC代码：
## Bellman：
```cpp
#include <cstdio>
#include <cstring>
using namespace std;

typedef long long LL;

struct Node {
    int u, v, w;
}edge[2500*10];

LL dist[1123];

int main() {
    int f;
    scanf("%d", &f);
    while (f--) {
        int n, m, w;
        scanf("%d %d %d", &n, &m, &w);
        for (int i = 0; i < m; i++) {
            int u, v, c;
            scanf("%d %d %d", &u, &v, &c);
            edge[i*2] = {u, v, c};
            edge[i*2+1] = {v, u, c};
        }
        m *= 2;
        for (int i = 0; i < w; i++) {
            int u, v, c;
            scanf("%d %d %d", &u, &v, &c);
            edge[i+m] = {u, v, -c};
        }

        bool flag = false;
        memset(dist, 0x3f, sizeof(dist));
        dist[1] = 0;
        for (int k = 0; k <= n; k++) {
            for (int j = 0; j < m + w; j++) {
                int u = edge[j].u;
                int v = edge[j].v;
                int w = edge[j].w;
                if (dist[v] > dist[u] + w) {
                    if (k == n)
                        flag = true;
                    dist[v] = dist[u] + w;
                }
            }
        }


        if (flag)
            printf("YES\n");
        else
            printf("NO\n");
    }
}
```

## SPFA：
```cpp
#include <cstdio>
#include <cstring>
#include <queue>
using namespace std;

typedef long long LL;


vector<pair<int, int> > edge[1123];
int dist[1123], cnt[1123];
bool vis[1123];

int main() {
    int f;
    scanf("%d", &f);
    while (f--) {
        int n, m, w;
        scanf("%d %d %d", &n, &m, &w);
        for (int i = 0; i <= n; i++) {
            edge[i].clear();
        }
        for (int i = 0; i < m; i++) {
            int u, v, c;
            scanf("%d %d %d", &u, &v, &c);
            edge[u].push_back(make_pair(v, c));
            edge[v].push_back(make_pair(u, c));
        }
        for (int i = 0; i < w; i++) {
            int u, v, c;
            scanf("%d %d %d", &u, &v, &c);
            edge[u].push_back(make_pair(v, -c));
        }

        bool flag = false;
        memset(dist, 0x3f, sizeof(dist));
        memset(cnt, 0, sizeof(cnt));
        queue<int> q;
        q.push(1);
        dist[1] = 0;
        vis[1] = true;

        while (!q.empty()) {
            int u = q.front();
            for (int i = 0; i < edge[u].size(); i++) {
                int v = edge[u][i].first;
                int w = edge[u][i].second;
                if (dist[v] > dist[u] + w) {
                    dist[v] = dist[u] + w;
                    if (++cnt[v] >= n) {
                        flag = true;
                        break;
                    }
                    if (!vis[v]) {
                        q.push(v);
                    }
                }
            }
            q.pop();
            vis[u] = false;
        }

        if (flag)
            printf("YES\n");
        else
            printf("NO\n");
    }
}
```