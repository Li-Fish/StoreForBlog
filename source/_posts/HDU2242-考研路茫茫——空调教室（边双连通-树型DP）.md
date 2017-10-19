---
title: HDU2242 - 考研路茫茫——空调教室（边双连通+树型DP）
date: 2017-08-03 10:38:56
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=2242

--------------------
# 题目大意：
给出一个连通的无向图，要求删除一条边使得无向图变为两个连通分量，每个点有一个权值$A_i$，使得两连通分量的权值差最小。

-------------------
# 解题过程：
拿来练手的题，看了一堆树和博客，这题主要是要处理重边比较麻烦，对于无向图的边双连通，其实和有向图的强连通的处理方式差不多。单纯的将边双连通分量标记好就可以了，每个点只属于一个边双连通分量。

另外比较重要是，对无向图进行双连通缩点后，所剩下的图是一个树形图！

--------------------
# 题目分析：
首先缩点成树形图，缩后点的权值为包含所有点的和。然后用树型DP求每个节点及其儿子的权值和。

这时对于树型图，剩下的每一条边都是桥。那么枚举每一个点，假设删去它与它父亲之间的边。那么拆成的两个连通分量的权值差为$ABS(\sum_{i=0}^{n}A_i - DP_i)$，$DP_i$为$i$节点及其儿子节点的权值和。这里缩点后是一个无根树，任意找一个点作为根节点即可。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 10000 + 10;

struct Edge {
    int v, nxt;
} edge[MAX << 2];

int n, m, data[MAX], sum;

int head[MAX], ecnt;
vector<int> G[MAX];

int dfs_clock, scc_cnt;
int low[MAX], pre[MAX], mark[MAX];
stack<int> S;

int dp[MAX];

void add_edge(int u, int v) {
    edge[ecnt].v = v;
    edge[ecnt].nxt = head[u];
    head[u] = ecnt++;
}

void init() {
    dfs_clock = scc_cnt = sum = ecnt = 0;
    memset(head, -1, sizeof(head));
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    memset(dp, 0, sizeof(dp));
    for (int i = 0; i <= n; i++) {
        G[i].clear();
    }

    for (int i = 0; i < n; i++) {
        scanf("%d", data + i);
        sum += data[i];
    }

    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        add_edge(u, v);
        add_edge(v, u);
    }
}


//无向图边双连通和有向图的强连通神似
void tarjan(int u, int fa) {
    low[u] = pre[u] = ++dfs_clock;
    S.push(u);

    for (int i = head[u]; ~i; i = edge[i].nxt) {
        //这里不直接用v != fa来做判断，是为了避免重边的情况
        if (i == (fa ^ 1)) continue;
        int v = edge[i].v;
        if (!pre[v]) {
            tarjan(v, i);
            low[u] = min(low[u], low[v]);
        }
        if (!mark[v]) {
            low[u] = min(low[u], pre[v]);
        }
    }
    if (pre[u] == low[u]) {
        scc_cnt++;
        int x;
        do {
            x = S.top(); S.pop();
            mark[x] = scc_cnt;
            dp[scc_cnt] += data[x];
        } while (x != u);
    }
}

void rebuild() {
    for (int u = 0; u < n; u++) {
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            if (mark[u] != mark[v]) G[mark[u]].push_back(mark[v]);
        }
    }
}

//树型DP求权值和
int dfs(int u, int fat) {
    for (int i = 0; i < G[u].size(); i++) {
        int v = G[u][i];
        if (v == fat) continue;
        dfs(v, u);
        dp[u] += dp[v];
    }
}

void solve() {
    for (int i = 0; i < n; i++) {
        if (!pre[i]) tarjan(i, -1);
    }
    if (scc_cnt == 1) { puts("impossible"); return; }
    rebuild();
    dfs(1, -1);
    int ans = 1e9;
    for (int i = 1; i <= scc_cnt; i++) {
        ans = min(ans, abs(sum - dp[i] * 2));
    }
    printf("%d\n", ans);
}

int main() {
    while (~scanf("%d %d", &n, &m) && n) {
        init();
        solve();
    }
}
```