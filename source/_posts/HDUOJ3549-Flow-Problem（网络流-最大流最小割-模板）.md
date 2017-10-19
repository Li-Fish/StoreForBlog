---
title: HDUOJ3549 - Flow Problem（网络流+最大流最小割+模板）
date: 2017-05-19 09:43:24
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=3549

----------------------------------
# 题目大意：
 有向图求最大流

--------------------------------
# 解题过程：


关于为什么增加流量时要增加一个反向负流量的边纠结了很久，最后还是想通了，其他就没难点了。

------------------------------
# 题目分析：

增广路算法，每一次尽可能的添加一条增广路，直到不能添加增广路为止，不过有个特殊的地方，按照挑战书上的说法是，可以把之前的流推回去。按照我的理解大概是现在的这个新加的增广路用之前流量的路径，然后把之前流的路径推回去，然后之前的流量寻找一个新的路径。




--------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 20, INF = 0x3f3f3f3f;

struct Node {
    //to表示要指向的点，cap是剩余流量，rev是指这条边的方向边是to这个点的第几条边
    int to, rev, cap;
    Node(int to, int cap, int rev):to(to), cap(cap), rev(rev) {}
};

vector<Node> edge[MAX];
int vis[MAX];

void add_edge(int u, int v, int w) {
    //添加边和反向边
    edge[u].push_back(Node(v, w, edge[v].size()));
    edge[v].push_back(Node(u, 0, edge[u].size()-1));
}

int dfs(int u, int end, int f) {
    if (u == end)
        return f;
    vis[u] = 1;
    for (int i = 0; i < edge[u].size(); i++) {
        Node& e = edge[u][i];
        if (!vis[e.to] && e.cap > 0) {
            int d = dfs(e.to, end, min(f, e.cap));
            //如果找到了增广路就返回
            if (d > 0) {
                //正向边容量减少的时候，反向边要增加容量
                e.cap -= d;
                edge[e.to][e.rev].cap += d;
                return  d;
            }
        }
    }
    return 0;
}

int max_flow(int s, int t) {
    int flow = 0;
    for (;;) {
        memset(vis, 0, sizeof(vis));
        int f = dfs(s, t, INF);
        //如果找不到增广路了，那么当前就是最大流了，返回
        if (f == 0)
            return flow;
        flow += f;
    }
}

int main() {
    int T;
    scanf("%d", &T);
    for (int cases = 1; cases <= T; cases++) {
        int n, m;
        scanf("%d %d", &n, &m);

        for (int i = 0; i <= n; i++)
            edge[i].clear();

        for (int i = 0; i < m; i++) {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            add_edge(u, v, w);
        }

        printf("Case %d: %d\n", cases, max_flow(1, n));
    }
}
```