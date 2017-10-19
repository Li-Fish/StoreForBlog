---
title: HDU1827 - Summer Holiday(强连通+缩点)
date: 2017-08-02 21:03:17
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=1827

--------------------
# 题目大意：
给出一个有向图，选择若干的点作为起点，使得从这些起点出发可以遍历所有的点，选出每个点为起点需要花费$A_i$，求最小费用。
$N <= 1000, M <= 2000$

-------------------

# 解题过程：

想起来还是比较容易的，因为在刷强连通的题，想到缩点转化成DAG就简单了。
不过卡了好久，因为自己的`++dfs_clock`写成`dfs_clock++`了。

--------------------
# 题目分析：

首先进行强连通分量分解，缩点化为DAG，因为强连通的每个点都可以互相到达，那么只需要能到达强连通分量里面的任一点就可以了，对于缩点后，他的费用为他包含的所有点中最小的。

然后现在是一个DAG图，如果一个点的入度不为0，那么他一定可以从另外的一个点到达，那么我只需要选取所有入度为0的点，并累加他们的值既可以了。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1000+10;
const int INF = 0x3f3f3f3f;

int n, m;

//G1为原图，G2为重建的图
vector<int> G1[MAX], G2[MAX];
//原始费用，重建的图的费用
int cost[MAX], ans_cost[MAX];

int low[MAX], dfn[MAX], mark[MAX], scc_cnt, dfs_clock;
int in[MAX], out[MAX];
stack<int> S;

void tarjan(int u) {
    low[u] = dfn[u] = ++dfs_clock;
    S.push(u);
    for (int i = 0; i < G1[u].size(); i++) {
        int v = G1[u][i];
        if (!dfn[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        }
        else if (!mark[v]) {
            low[u] = min(low[u], dfn[v]);
        }
    }
    if (low[u] == dfn[u]) {
        int x;
        ++scc_cnt;
        do {
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
        } while (x != u);
    }
}

void rebuild() {
    //枚举每条边进行重建图
    for (int u = 1; u <= n; u++) {
        //更新强连通分量的花费
        ans_cost[mark[u]] = min(ans_cost[mark[u]], cost[u]);
        for (int i = 0; i < G1[u].size(); i++) {
            int v = G1[u][i];
            if (mark[u] != mark[v]) {
                in[mark[v]]++;
                G2[mark[u]].push_back(mark[v]);
            }
        }
    }
}

void init() {
    memset(dfn, 0, sizeof(dfn));
    memset(ans_cost, INF, sizeof(ans_cost));
    memset(mark, 0, sizeof(mark));
    memset(in, 0, sizeof(in));
    memset(out, 0, sizeof(out));
    scc_cnt = dfs_clock = 0;
    for (int i = 1; i <= n; i++) {
        G1[i].clear(); G2[i].clear();
        scanf("%d", cost + i);
    }
    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        G1[u].push_back(v);
    }
}

void solve() {
    for (int i = 1; i <= n; i++) {
        if (!dfn[i]) tarjan(i);
    }

    rebuild();

    int ans = 0, ans_cnt = 0;
    //统计入度为0的点
    for (int i = 1; i <= scc_cnt; i++) {
        if (in[i] == 0) ans += ans_cost[i], ans_cnt++;
    }
    printf("%d %d\n", ans_cnt, ans);
}

int main() {

    while (~scanf("%d %d", &n, &m)) {
        init();
        solve();
    }
}
```