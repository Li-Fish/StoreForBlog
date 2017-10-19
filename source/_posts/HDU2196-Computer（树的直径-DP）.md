---
title: HDU2196 - Computer（树的直径+DP）
date: 2017-06-28 20:41:14
categories: [ACM, DP, 树型DP]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=2196

-----------------------------
# 题目大意：
给定一个无向连通无环图，求每个节点到达的最远的节点的距离。


----------------------------
# 解题过程：
上午看了一下DP进阶之路的PDF，突然想学树型DP。然后找到了这个题，之前做了一个POJ的BFS求树的直径的，这次再来一发DP的。

--------------------------------
# 题目分析：
由于题目给的是无向连通无环图，这里构造出一颗树来，不妨假设节点$1$为根。

那么对于任意一个节点，他能到达的最远的节点一定是他子树中的一个节点，或者经过他父亲到达的一个节点。

这里可以经过一次DFS求出每个节点到他子树中最远的节点的距离。

然后第二次DFS的时候求出每个节点经过他根节点能到达的最远的距离，求经过根节点能到达的最远的距离时，需要当前节点到根节点的距离加上根节点到最远的节点的距离。不过要注意的是，可能当前节点在根节点最远距离的路径上。

这里解决的方法是，求出每个节点最远的距离和次远的距离，如果当前节点在最远的距离路径上那么就选择次远的距离，判断是否在路径上需要记录下最远的距离是由那个儿子求出来的。

----------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 11234;
const int INF = 0x3f3f3f3f;

int n;
vector<pair<int, int> > edge[MAX];

//dis1记录最远距离，dis2记录次远距离，num用来标记最远距离由那个节点而来的
int num[MAX], dis1[MAX], dis2[MAX], dp[MAX];


void read() {
    for (int i = 1; i <= n; i++) {
        edge[i].clear();
    }
    memset(dp, 0, sizeof(dp));
    for (int i = 2; i <= n; i++) {
        int v, w;
        scanf("%d %d", &v, &w);
        edge[v].push_back(make_pair(i, w));
    }
}

void dfs1(int u) {
    dis1[u] = 0;
    //这里的含义是，对于每个节点求他所有儿子节点的最远距离，那么当前节点的最远距离就是到儿子节点的距离加上儿子节点的最远距离
    //显然对于一个叶子节点，最远距离是0
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i].first;
        int w = edge[u][i].second;
        dfs1(v);
        if (dis1[v] + w > dis1[u]) {
            dis1[u] = dis1[v] + w;
            num[u] = v;
        }
    }
    //和上一步相似，求次远距离
    dis2[u] = -INF;
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i].first;
        int w = edge[u][i].second;
        //当前距离不能是最远距离
        if (v != num[u] && dis1[v] + w > dis2[u]) {
            dis2[u] = dis1[v] + w;
        }
    }
}

void dfs2(int u) {
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i].first;
        int w = edge[u][i].second;
        //dp[u]的含义是，当前节点经过父节点能到达的最远的距离
        //如果当前节点在最远距离的路径上，那么用次远距离，否则用最远距离
        if (v == num[u])
            dp[v] = max(dp[u], dis2[u]) + w;
        else
            dp[v] = max(dp[u], dis1[u]) + w;
        dfs2(v);
    }
}

void go() {
    dfs1(1);
    dfs2(1);
    //输出每个点的最远距离
    for (int i = 1; i <= n; i++) {
        printf("%d\n", max(dp[i], dis1[i]));
    }
}

int main() {
    while (~scanf("%d", &n)) {
        read();
        go();
    }
}
```