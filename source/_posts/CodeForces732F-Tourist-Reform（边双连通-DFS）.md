---
title: CodeForces732F - Tourist Reform（边双连通 + DFS）
date: 2017-10-15 12:33:53
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：

http://codeforces.com/contest/732/problem/F



# 题目大意：

给出一张 n 个顶点， m 条边的无向图，保证图连通，没有重边，现在给每个边加上方向，记从点 i 出发可以访问到的点的数量为 $r_i$，求一种分配方向的方式，使得最小的 $r_i$ 尽量的大。

$2 \le n \le 400000, 1 \le m \le 400000$

# 题目分析：

这里就引用下 [dalao的博客](https://blog.sengxian.com/solutions/cf-732f):

> 我们考虑如何将边定向，定向成 DAG 肯定是极不好的，因为 DAG 里边存在没有出度的点，而这样的话，答案就必然为 1 了。也就是说，不能出去的点，最好要形成一个环，这样答案就是环的大小了。
>
> 将图分解为若干边-双连通分量，将每个边-双连通分量看作一个点，那么此时形成了一棵缩点树。对于每个边-双连通分量，我们可以将里边的边定向，使之成为强联通分量。再将缩点后的树边定向，成为一个边指向根的树形图，这样答案就是根代表的边-双连通分量的答案，由于任意点都可以做树形图的根，所以答案就是最大的边-双连通分量的大小。
>
> **定理：**答案就是是最大的边-双连通分量的大小。
> **证明：**前面已经证明了，最大的边-双连通分量的大小是一个合法答案。现在只需证明，最大的答案不会大于最大的边-双连通分量的大小：考虑定向后的图，将其强联通缩点，答案就是没有出度的强联通分量中最小的那个，如果这个值比最大的边-双连通分量的大小更大，那么考虑将这个强联通分量中的边改为无向边，这就能形成一个边-双连通分量，而且比原图中最大的边-双连通分量的大小还要大，这就产生了矛盾。
>
> 考虑输出方案，树边是很好定向的，DFS 一下缩点树就行了。如何将边-双连通分量中的边定向，使得形成一个强联通分量呢？我们考虑直接使用 DFS 中第一次访问边的顺序，为什么？因为利用这个顺序，肯定能保证联通。我们再考虑 DFS 树，在 DFS 树中，每个点一定能通过 DFS 返祖边和非返祖边的结合，走到自己上方的点（否则存在桥，与边-双连通的定义违背），所以每个点都可以通过定向后的边走到根，自然证明这个原先的边-双连通分量通过这种定向方法后强联通。


# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 4112345;

struct Edge {
    int u, v, nxt;
} edge[MAX << 1];

int n, m;

int head[MAX], ecnt;

int dfs_clock, scc_cnt;
int low[MAX], pre[MAX], mark[MAX];
stack<int> S;

int num[MAX], ans, id;
int data[MAX][2];

void add_edge(int u, int v) {
    edge[ecnt].u = u;
    edge[ecnt].v = v;
    edge[ecnt].nxt = head[u];
    head[u] = ecnt++;
}

void init() {
    dfs_clock = scc_cnt = ecnt = 0;
    memset(head, -1, sizeof(head));
    memset(mark, 0, sizeof(mark));

    scanf("%d %d", &n, &m);
    for (int i = 0; i < m; i++) {
        scanf("%d %d", &data[i][0], &data[i][1]);
        add_edge(data[i][0], data[i][1]);
        add_edge(data[i][1], data[i][0]);
    }
}

void tarjan(int u, int fa) {
    low[u] = pre[u] = ++dfs_clock;
    S.push(u);

    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (v == fa) continue;
        if (!pre[v]) {
            tarjan(v, u);
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
            x = S.top();
            S.pop();
            mark[x] = scc_cnt;
            num[scc_cnt]++;
            if (num[scc_cnt] > ans) {
                ans = num[scc_cnt];
                id = u;
            }
        } while (x != u);
    }
}

bool vis[MAX];
int fa[MAX];

void dfs1(int u) {
    fa[u] = true;
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (vis[i / 2]) continue;
        if (mark[v] != mark[u]) continue;
        vis[i / 2] = true;
        data[i / 2][0] = u;
        data[i / 2][1] = v;
        if (fa[v]) continue;
        dfs1(v);
    }
}

void dfs2(int u) {
    vis[u] = true;
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        //这里类似 DFS 一棵树的过程，如果当前双连通分量已经访问过了，那么不应该通过其他强连通分量去访问了
        if (mark[v] != mark[u]) {
            if (fa[mark[v]] == 0 || fa[mark[v]] == mark[u]) {
                fa[mark[v]] = mark[u];
                data[i/2][0] = v;
                data[i/2][1] = u;
            }
        }
        if (vis[v]) continue;
        dfs2(v);
    }
}

void solve() {
    tarjan(1, -1);

    //第一次 DFS 为双连通分量里面的边分配方向
    for (int i = 1; i <= n; i++) {
        dfs1(i);
    }

    //第二次为连接不同的双连通分量的边分配方向
    memset(vis, 0, sizeof(vis));
    memset(fa, 0, sizeof(fa));
    fa[mark[id]] = -1;
    dfs2(id);

    printf("%d\n", ans);
    for (int i = 0; i < m; i++) {
        printf("%d %d\n", data[i][0], data[i][1]);
    }
}

int main() {
    init();
    solve();
}
```
# 解题过程：

这题卡了好久，主要是为缩点后的图重新建边后会爆内存，然后只能用第一次建的边去 DFS，然后就写的有点恶心了...