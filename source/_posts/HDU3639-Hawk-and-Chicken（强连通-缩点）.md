---
title: HDU3639 - Hawk-and-Chicken（强连通+缩点）
date: 2017-08-03 08:49:59
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=3639

--------------------
# 题目大意：
给出一个有向图，记每个点的值为他的子节点的个数，求出最大的值为多少，有那些顶点值最大。

-------------------
# 解题过程：
题意刚开始理解错了...以为一个点可以贡献多次。

--------------------
# 题目分析：
这题写出来主要是提醒自己一下，在DAG上不能直接用一个点维护他所有子节点权值和那样的东西，因为DAG上一个点可以有多个父亲，不像树形图那样。

另外这个题，只需要统计入度为0的点就可以了，刚开始要方向建图，这样dfs的时候才符合他的子节点贡献给父亲节点。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 5000 + 10;

int T, n, m, Case;

vector<int> G1[MAX], G2[MAX];

int pre[MAX], low[MAX], mark[MAX], dfs_clock, scc_cnt;
stack<int> S;

int self_supports[MAX], dp[MAX], in[MAX];
bool winner[MAX];

void tarjan(int u) {
    pre[u] = low[u] = ++dfs_clock;
    S.push(u);
    for (int i = 0; i < G1[u].size(); i++) {
        int v = G1[u][i];
        if (!pre[v]) {
            tarjan(v);
            low[u] = min(low[u], low[v]);
        }
        if (!mark[v]) {
            low[u] = min(low[u], pre[v]);
        }
    }
    if (low[u] == pre[u]) {
        scc_cnt++;
        int x;
        do {
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
        } while (x != u);
    }
}

void get_G() {
    for (int i = 1; i <= n; i++) {
        self_supports[mark[i]]++;
    }
    for (int u = 1; u <= n; u++) {
        for (int i = 0; i < G1[u].size(); i++) {
            int v = G1[u][i];
            if (mark[u] != mark[v]) {
                G2[mark[u]].push_back(mark[v]);
                in[mark[v]]++;
            }
        }
    }
}

void init() {
    scanf("%d %d", &n, &m);
    dfs_clock = scc_cnt = 0;
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    memset(winner, 0, sizeof(winner));
    memset(dp, -1, sizeof(dp));
    memset(in, 0, sizeof(in));
    memset(self_supports, 0, sizeof(self_supports));
    for (int i = 0; i <= n; i++) {
        G1[i].clear();
        G2[i].clear();
    }
    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        u++, v++;
        G1[v].push_back(u);
    }
}

int dfs(int u) {
    int ans = self_supports[u] - 1;
    pre[u] = 1;
    for (int i = 0; i < G2[u].size(); i++) {
        int v = G2[u][i];
        if (!pre[v]) {
            ans += dfs(v) + 1;
        }
    }
    return ans;
}

void solve() {
    for (int i = 1; i <= n; i++) {
        if (!pre[i]) tarjan(i);
    }
    get_G();
    int ans = 0;
    for (int i = 1; i <= scc_cnt; i++) {
        if (in[i] != 0) continue;
        memset(pre, 0, sizeof(pre));
        dp[i] = dfs(i);
        ans = max(dp[i], ans);
    }
    for (int i = 1; i <= scc_cnt; i++) {
        if (dp[i] == ans) winner[i] = true;
    }
    bool fir = true;
    printf("Case %d: %d\n", ++Case, ans);
    for (int i = 1; i <= n; i++) {
        if (winner[mark[i]]) {
            if (!fir) putchar(' ');
            printf("%d", i - 1);
            fir = false;
        }
    }
    putchar('\n');
}

int main() {
    scanf("%d", &T);
    while (T--) {
        init();
        solve();
    }
}
```