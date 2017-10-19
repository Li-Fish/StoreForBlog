---
title: POJ3169 - Layout （差分约束）
date: 2017-05-15 11:12:44
categories: [ACM, 图论, 差分约束]
tags:
---
# 题目链接：
https://cn.vjudge.net/problem/POJ-3169

------------------------------
# 题目大意：
![这里写图片描述](http://img.blog.csdn.net/20170515103745320?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

-----------------------------------
# 解题过程：
 刚开始是一脸懵逼的，怎么还有这种题，完全没想法。
 然后看书上说是差分约束，然后和最短路类比了下，还是没看懂，最后是去网上搜了下博客，然后又自己画了画才弄懂。

 http://www.cppblog.com/menjitianya/archive/2015/11/19/212292.html

--------------------------------
# 题目分析：

 关于最短路，设函数 ***d(v)*** 的含义是从起点到节点 v 的最短距离，那么对于任意权值为 ***w*** 的边 ***e = (u, v)*** ，都有 ***d(u) + w >= d(v)*** 成立，公式可化为 ***w >= d(v) - d(u)*** 。最短路就是求解这组方程，变量是 ***d(v/x)*** ，求解这组方程就是意味着求出满足方程的 ***d(v) - d(u)*** 的最大值。

 对于这个题目，有三个约束条件，第一个是牛按编号顺序排，第二个是某两头牛之间的距离不能大于一个值，第三个是某两头牛之间的距离必须不小于一个值。

 假设这两头牛分别是 ***u*** 和 ***v*** ，***(v > u)***，约束距离是 ***w*** ，对于第一个条件， ***d(v) - d(u) >= 0*** 即 ***d(v) + 0 >= d(u)*** ，从***v***向***u***建一条权值为 ***0*** 的边。

 对于第二个条件，***d(u) + w >= d(v)***，即从 ***u*** 向 ***v*** 建一条权值为 ***w*** 的边。

 对于第三个条件，***d(u) + w <= d(v)***， ***d(v) - w >= d(u)***，从 ***v*** 向 ***u*** 建一条权值为 ***-w*** 的边。

 然后对于上面所建立的图，求出从起点到节点的最短距离即是答案。

 如果这个图无法联通，表示没有可行的方案。如果存在负环，那么这个距离是可以无限大的。


# AC代码：

```cpp
#include <vector>
#include <cstring>
#include <queue>
#include <cstdio>
using namespace std;

const int MAX = 11234, INF = 0x3f3f3f3f;
int frequency[MAX], dist[MAX], vis[MAX];
int N, ML, MD;
vector<pair<int, int> > edge[MAX];

int spfa() {
    memset(dist, INF, sizeof(dist));
    queue<int> q;
    q.push(1);
    dist[1] = 0;

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        vis[u] = 0;

        for (int i = 0; i < edge[u].size(); i++) {
            int v = edge[u][i].first;
            int w = edge[u][i].second;

            if (dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                if (!vis[v]) {
                    vis[v] = 1;
                    frequency[v]++;
                    if (frequency[v] > N)
                        return -INF;
                    q.push(v);
                }
            }
        }
    }

    return dist[N];
}

int main() {
    scanf("%d %d %d", &N, &ML, &MD);

    int u, v, w;
    for (int i = 2; i <= N; i++) {
        edge[i].push_back(make_pair(i-1, 0));
    }
    for (int i = 0; i < ML; i++) {
        scanf("%d %d %d", &u, &v, &w);
        edge[u].push_back(make_pair(v, w));
    }
    for (int i = 0; i < MD; i++) {
        scanf("%d %d %d", &u, &v, &w);
        edge[v].push_back(make_pair(u, -w));
    }

    int ans = spfa();
    if (ans <= -INF) {
        printf("-1\n");
    }
    else if (ans >= INF) {
        printf("-2\n");
    }
    else {
        printf("%d\n", ans);
    }
}

```
