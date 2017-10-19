---
title: POJ1904 - King's Quest & HDU4685 - Prince and Princess（强连通 + 二分图匹配）
date: 2017-08-18 11:47:34
categories: [ACM, 图论, 连通性]
tags:
---
# 题目链接：

http://poj.org/problem?id=1904

http://acm.hdu.edu.cn/showproblem.php?pid=4685



--------------------
# 题目大意：

#### POJ1904： 

一个国王有 N 个儿子，并且有 N 个公主，每个王子可以和一些喜欢公主结婚。公主没有限制。现在给出了一个 王子-公主 的完美匹配。

现在问每个王子，可以和那些公主结婚，结婚后其他的王子仍然可以完美匹配。



#### HDU4685:

题意和上面一样，不过没给出初始的匹配，不保证完美匹配，公主和王子的数量也有可能不同。



-------------------
# 解题过程：

一开始完全没思路，然后去翻了一堆的博客才看懂。



--------------------
# 题目分析：

首先以王子和公主为点建图，对每个王子向他们喜欢的公主连一条有向边。

然后进行二分图匹配，对一堆匹配的 王子-公主 从公主向王子建一条有向边。

进行强连通缩点，每个王子对于属于同一强连通分量中喜欢的公主都可以选择。



为什么这样就这样呢？可以类比网络流的退流或者匈牙利的思想，当一王子选择强连通里的非匹配的公主的话，那么这个公主的原配一定也可以选择另一个公主，并且一定可以经过若干次换妻 play 使得强连通分量内的所有王子都有匹配的公主，这里就不严格的证明了。



对于第二个题，可能有一些王子或公主一开始就没有匹配，但是他按题意应该是可以选择一些公主或者王子并不影响其他人匹配的。

这时候我们对没有匹配的公主增加一个虚拟的王子并匹配建边，并且这个王子喜欢所有的公主。对于没有匹配的王子新增一个虚拟的公主来匹配，并且这个公主被所有王子喜欢。

为什么要喜欢所有的公主或者被所有王子喜欢，是为了让这个虚拟王子/公主有机会参加所有人的换妻 play 中，注意这里虚拟节点没必要和虚拟节点建边。



----------------------
# AC代码：



####POJ1904：

```cpp
#include <cstdio>
#include <cstring>
#include <stack>
#include <vector>
#include <algorithm>

using namespace std;

const int MAXM = 500000 + 10;
const int MAXN = 4000 + 10;

struct Edge {
    int u, v, nxt;
} edge[MAXM];

int head[MAXN], etot;


//本题的算法复杂度是 O(N+M) 的，读入数据也是 O(N+M) 对总时间影响较大
//用输入挂加速后由 9s 变成了 500ms
int Scan() {
    int res = 0, ch, flag = 0;
    if ((ch = getchar()) == '-')
        flag = 1;
    else if (ch >= '0' && ch <= '9')
        res = ch - '0';
    while ((ch = getchar()) >= '0' && ch <= '9')
        res = res * 10 + ch - '0';
    return flag ? -res : res;
}

void Out(int a) {
    if (a > 9)
        Out(a / 10);
    putchar(a % 10 + '0');
}

void add_edge(int u, int v) {
    edge[etot].v = v;
    edge[etot].u = u;
    edge[etot].nxt = head[u];
    head[u] = etot++;
}

int n;

int pre[MAXN], low[MAXN], mark[MAXN], dfs_clock, scc_cnt;
stack<int> S;

void dfs(int u) {
    low[u] = pre[u] = ++dfs_clock;
    S.push(u);
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (!pre[v]) {
            dfs(v);
            low[u] = min(low[u], low[v]);
        } else if (!mark[v]) {
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

void tarjan() {
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    for (int i = 1; i <= n; i++) {
        if (!pre[i]) dfs(i);
    }
}

void init() {
    memset(head, -1, sizeof(head));
    etot = 0;
    n = Scan();
    for (int u = 1; u <= n; u++) {
        int k;
        k = Scan();
        while (k--) {
            int v;
            v = Scan();
            add_edge(u, v + n);
        }
    }
    for (int u = 1; u <= n; u++) {
        int v;
        v = Scan();
        add_edge(v + n, u);
    }
}

void solve() {
    tarjan();

    vector<int> data;
    for (int u = 1; u <= n; u++) {
        data.clear();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            //如果属于同一强连通分量，那么可以选择
            if (mark[u] == mark[v]) data.push_back(v);
        }
        Out(data.size());
        //对答案排序
        sort(data.begin(), data.end());
        for (int i = 0; i < data.size(); i++) {
            putchar(' ');
            Out(data[i] - n);
        }
        putchar('\n');
    }
}

int main() {
    init();
    solve();
}
```
####HDU4685

```cpp
#include <cstdio>
#include <cstring>
#include <stack>
#include <vector>
#include <algorithm>

using namespace std;

const int MAXM = 500000 + 10;
const int MAXN = 4000 + 10;

struct Edge {
    int u, v, nxt;
} edge[MAXM];

int head[MAXN], etot;
int n, m, Case, tot;

int pre[MAXN], low[MAXN], mark[MAXN], dfs_clock, scc_cnt;
stack<int> S;

int matching[MAXN];

int Scan() {
    int res = 0, ch, flag = 0;
    if ((ch = getchar()) == '-')
        flag = 1;
    else if (ch >= '0' && ch <= '9')
        res = ch - '0';
    while ((ch = getchar()) >= '0' && ch <= '9')
        res = res * 10 + ch - '0';
    return flag ? -res : res;
}

void Out(int a) {
    if (a > 9)
        Out(a / 10);
    putchar(a % 10 + '0');
}

void add_edge(int u, int v) {
    edge[etot].v = v;
    edge[etot].u = u;
    edge[etot].nxt = head[u];
    head[u] = etot++;
}

void dfs(int u) {
    low[u] = pre[u] = ++dfs_clock;
    S.push(u);
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (!pre[v]) {
            dfs(v);
            low[u] = min(low[u], low[v]);
        } else if (!mark[v]) {
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

void tarjan() {
    memset(pre, 0, sizeof(pre));
    memset(mark, 0, sizeof(mark));
    for (int i = 1; i <= tot; i++) {
        if (!pre[i]) dfs(i);
    }
}

bool find(int u) {
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (pre[v]) continue;
        pre[v] = true;
        if (matching[v] == -1 || find(matching[v])) {
            matching[v] = u;
            matching[u] = v;
            return true;
        }
    }
    return false;
}

void match() {
    memset(matching, -1, sizeof(matching));
    for (int i = 1; i <= n; i++) {
        memset(pre, 0, sizeof(pre));
        find(i);
    }
}

void init() {
    memset(head, -1, sizeof(head));
    etot = 0;
    n = Scan();
    m = Scan();
    for (int u = 1; u <= n; u++) {
        int k;
        k = Scan();
        while (k--) {
            int v;
            v = Scan();
            add_edge(u, v + n);
        }
    }
}

void solve() {
    match();

    tot = n + m;
    for (int i = 1; i <= n; i++) {
        //如果当前节点未匹配
        if (matching[i] == -1) {
            //增加一个虚拟公主，并建边
            ++tot;
            add_edge(tot, i);
            for (int j = 1; j <= n; j++) {
                add_edge(j, tot);
            }
        } else {
            //如果已匹配，建一条反向边
            add_edge(matching[i], i);
        }
    }
    for (int i = n + 1; i <= n + m; i++) {
        //同理增加一个虚拟公主
        if (matching[i] == -1) {
            ++tot;
            add_edge(i, tot);
            for (int j = n + 1; j <= n + m; j++) {
                add_edge(tot, j);
            }
        }
    }

    tarjan();

    printf("Case #%d:\n", ++Case);
    vector<int> data;
    for (int u = 1; u <= n; u++) {
        data.clear();
        for (int i = head[u]; ~i; i = edge[i].nxt) {
            int v = edge[i].v;
            //如果是虚拟节点就忽略
            if (v > n + m) continue;
            //如果和公主属于同一强连通分量，那么可以选择
            if (mark[u] == mark[v]) data.push_back(v);
        }
        Out(data.size());
        sort(data.begin(), data.end());
        for (int i = 0; i < data.size(); i++) {
            putchar(' ');
            Out(data[i] - n);
        }
        putchar('\n');
    }
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        init();
        solve();
    }
}
```

