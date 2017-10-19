---
title: HDU3861 - The King’s Problem（强连通+缩点+匈牙利）
date: 2017-08-02 21:28:16
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：
--------------------
http://acm.hdu.edu.cn/showproblem.php?pid=3861

# 题目大意：

给出一个有向图，想在要把无向图分成k个部分，对于每对可以互相到达的(u, v)要属于同一个部分，另外每个部分中任意两个点(u, v)至少可以由u到达v或者由v到达u。

-------------------
# 解题过程：

刚开始一点都不会，还看错了题意，只知道可以缩成DAG，但是缩完也不知道该怎么搞，只好去翻了下博客，发现缩完点，要求的就是最小路径覆盖，然后最小路径覆盖需要用到二分图匹配，顺手去学了一下匈牙利。

不过匈牙利刚开始没仔细看，学的曲线有点歪...回来才弄明白。

--------------------
# 题目分析：

首先对原图强连通分量分解，缩点转化为DAG图。然后分析一下，题目说描述的就是最小路径覆盖。

> 一个有向图中，路径覆盖就是在图中找一些路径，使之覆盖了图中的所有顶点，且任何一个顶点有且只有一条路径与之关联

对于DAG的最小路径覆盖就是DAG的顶点数减去DAG对应的二分图的最大匹配数。

这篇[博客](http://www.cnblogs.com/justPassBy/p/5369930.html) 讲的就非常好，也有必要的证明。

需要补充一下的就是代码部分了，这个代码写的有点强，虽然说上面的分析讲到要对DAG的每个点拆成两个点建图，然后跑最大二分图匹配。但是代码上并没有拆点。

使用匈牙利匹配的时候，DFS函数里面从u匹配到v时，只对v进行标记一下，这样虽然u已经匹配了，但是他还可以当做入点再匹配一次。这样也相当于拆点建图了。对于每个点当做入点被匹配和当做出点被匹配是相对独立的，正好符合上面描述的算法。

不过这里老老实实的拆点再匹配也可以AC，就是相对于上面的代码麻烦点了。

----------------------
# AC代码：

## 不拆点
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 5000 + 10;

vector<int> G1[MAX], G2[2 * MAX];

int n, m, T;

int pre[MAX], low[MAX], mark[MAX], dfs_clock, scc_cnt;
stack<int> S;

int check[2 * MAX], matching[2 * MAX];

void tarjan(int u) {
    low[u] = pre[u] = ++dfs_clock;
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
        ++scc_cnt;
        int x;
        do {
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
        } while (x != u);
    }
}

void init() {
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    scc_cnt = dfs_clock = 0;
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= n; i++) G1[i].clear();
    for (int i = 1; i <= 2 * n; i++) G2[i].clear();
    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        G1[u].push_back(v);
    }
}

void rebuild() {
    for (int u = 1; u <= n; u++) {
        for (int i = 0; i < G1[u].size(); i++) {
            int v = G1[u][i];
            if (mark[u] != mark[v]) {
                G2[mark[u]].push_back(mark[v]);
            }
        }
    }
}

bool dfs(int u) {
    for (int i = 0; i < G2[u].size(); i++) {
        int v = G2[u][i];
        if (!check[v]) {
            check[v] = true;
            if (matching[v] == -1 || dfs(matching[v])) {
                matching[v] = u;
                return true;
            }
        }
    }
    return false;
}

void solve() {
    for (int i = 1; i <= n; i++) {
        if (!pre[i]) tarjan(i);
    }

    memset(pre, 0, sizeof(pre));
    rebuild();

    int max_match = 0;
    memset(matching, -1, sizeof(matching));
    for (int i = 1; i <= scc_cnt; i++) {
        if (matching[i] == -1) {
            memset(check, 0, sizeof(check));
            if (dfs(i)) max_match++;
        }
    }

    printf("%d\n", scc_cnt - max_match);
}

int main() {
    scanf("%d", &T);
    while (T--) {
        init();
        solve();
    }
}
```

## 拆点
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 5000 + 10;

vector<int> G1[MAX], G2[2 * MAX];

int n, m, T;

int pre[MAX], low[MAX], mark[MAX], dfs_clock, scc_cnt;
stack<int> S;

int check[2 * MAX], matching[2 * MAX];

void tarjan(int u) {
    low[u] = pre[u] = ++dfs_clock;
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
        ++scc_cnt;
        int x;
        do {
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
        } while (x != u);
    }
}

void init() {
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    scc_cnt = dfs_clock = 0;
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= n; i++) G1[i].clear();
    for (int i = 1; i <= 2 * n; i++) G2[i].clear();
    for (int i = 0; i < m; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        G1[u].push_back(v);
    }
}

void rebuild() {
    for (int u = 1; u <= n; u++) {
        for (int i = 0; i < G1[u].size(); i++) {
            int v = G1[u][i];
            if (mark[u] != mark[v]) {
                G2[mark[u]].push_back(mark[v] + scc_cnt);
            }
        }
    }
}

bool dfs(int u) {
    for (int i = 0; i < G2[u].size(); i++) {
        int v = G2[u][i];
        if (!check[v]) {
            check[v] = true;
            if (matching[v] == -1 || dfs(matching[v])) {
                matching[v] = u;
                matching[u] = v;
                return true;
            }
        }
    }
    return false;
}

void solve() {
    for (int i = 1; i <= n; i++) {
        if (!pre[i]) tarjan(i);
    }

    memset(pre, 0, sizeof(pre));
    rebuild();

    int max_match = 0;
    memset(matching, -1, sizeof(matching));
    for (int i = 1; i <= 2 * scc_cnt; i++) {
        if (matching[i] == -1) {
            memset(check, 0, sizeof(check));
            if (dfs(i)) max_match++;
        }
    }

    printf("%d\n", scc_cnt - max_match);
}

int main() {
    scanf("%d", &T);
    while (T--) {
        init();
        solve();
    }
}
```