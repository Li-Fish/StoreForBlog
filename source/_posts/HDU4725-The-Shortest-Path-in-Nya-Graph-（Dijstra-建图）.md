---
title: HDU4725 - The Shortest Path in Nya Graph （Dijstra + 建图）
date: 2017-05-22 11:01:02
categories: [ACM, 图论, 最短路]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=4725


--------------------------
# 题目大意：

 现在有N个节点，编号从1到N。有M条权值为Ci无向边，链接着两个节点。
 新加入了一个条件，每个节点在一个层内，假设在 x 层，那么在 x 层内的节点可以直接到达 x + 1 层或 x -1 层的任意节点，花费为 C 。

 现在求从 1 到 N 的最短路。

------------------------------
# 解题过程：
 
 比赛的时候没做出来，现在才开始补题，当时觉得挺难的，没想到拆点重新建图。想通了就觉得挺简单了，然后处理下细节，不过这个题卡 SPFA 有点坑。

-------------------------
# 题目分析：

 对于每一层，加入两个点，一个入点，一个出点。入点和出点和这一层里的所有点连一条边，并且权值为0 ， 然后每一个出点和相邻的两层的入点相连。剩下的就是普通的最短路了。

 关于为什么要建一个入点一个出点而不是只加入一个点，是为了防止同一层的点互相移动，这样同一层里的点的最短距离都变成 0 了。

--------------------------------------
# AC代码：
```cpp
#include <cstdio>
#include <vector>
#include <queue>
#include <cstring>
using namespace std;

const int MAX = 312345, INF = 0x3f3f3f3f;

int N, M, C;

vector<pair<int, int> > edge[MAX];
int vis[MAX], dist[MAX];

int dijstra() {
    //普通的dijstra求最短路
    memset(dist, INF, sizeof(dist));
    priority_queue<pair<int, int> > q;
    q.push(make_pair(0, 1));
    dist[1] = 0;
    while (!q.empty()) {
        int u = q.top().second;
        q.pop();
        for (int i = 0; i < edge[u].size(); i++) {
            int v = edge[u][i].first;
            int w = edge[u][i].second;
            if (dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                q.push(make_pair(-dist[v], v));
            }
        }
    }
    return dist[N];
}

int main() {
    int T, cases = 0;
    scanf("%d", &T);
    while (T--) {

        for (int i = 0; i <= N*3; i++) {
            edge[i].clear();
            vis[i] = 0;
        }

        scanf("%d %d %d", &N, &M, &C);
        for (int i = 1; i <= N; i++) {
            int t;
            scanf("%d", &t);
            vis[t] = 1;
            //这里 t*2-1+N 代表第 t 层的入点， t*2+N 代表出点
            edge[t*2-1+N].push_back(make_pair(i, 0));
            edge[i].push_back(make_pair(t*2+N, 0));
        }

        for (int i = 0; i < M; i++) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            edge[u].push_back(make_pair(v, w));
            edge[v].push_back(make_pair(u, w));
        }

        for (int i = 1; i <= N; i++) {
            int u = N + i*2;

            //当前层的出点与相邻层的入点建边
            if (i > 1) {
                int v = u-3;
                edge[u].push_back(make_pair(v, C));
            }
            if (i < N) {
                int v = u+1;
                edge[u].push_back(make_pair(v, C));
            }
        }

        int ans = dijstra();
        printf("Case #%d: %d\n", ++cases, ans == INF? -1:ans);
    }
}
```
