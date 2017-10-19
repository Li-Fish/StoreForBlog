---
title: FZU - 2112 Tickets （欧拉回路+联通块判断）
date: 2017-05-04 17:38:11
categories: [ACM, 图论]
tags:
---
# 题目链接：
https://cn.vjudge.net/problem/FZU-2112

-------------------
# 题目大意：
给定一个图的N个边，求添加最少的边使这个图形成欧拉路。不必所有的点都联通，只需要把已给出的边形成欧拉路。

---------------------------------------
# 解题过程：

之前好像做过一次做过题，然后又看到了，当时也忘记在哪看到了，不知道A了没有，反正记得看题解没理解……

这道题刚开始理解错了，还以为是求哈密顿图，这个是真的不会。回来又看了一遍题意才发现是之前做过的欧拉路。

然后又仔细想了下，可算是想通了。

-----------------------

# 题目分析：
首先BFS一遍记录下联通块个数， 每个联通块奇点的个数。

要构成欧拉路，就要把这些联通块之前用一条边连起来。
假设N个联通块，那么需要N-1个边把他们连接起来。

然后对于每一个联通块，如果奇点个数大于2，那么每多了个奇点，需要花费一条边连起来这两个奇点。

可以证明，对于任意一个无向图，他奇点的个数一定是偶数个。对于每无向边那么一定会给两个点增加一个度，所以奇点的个数一定是偶数个的。





----------------
# AC代码：

```cpp
#include<cstdio>
#include<cstring>
#include<vector>
using namespace std;

const int MAX = 112345;

int n, m;
int vis[MAX];
int book[MAX];
vector<int> edge[MAX];

//Dfs判断联通块个数和统计奇点的个数
void dfs(int u, int& odd) {
    if (edge[u].size()%2)
        odd++;
    vis[u] = 1;
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i];
        if (!vis[v])
            dfs(v, odd);
    }
}

int solve() {
    int cnt = 0;
    for (int i = 1; i <= n; i++) {
        if (vis[i] || !book[i])
            continue;
        int odd = 0;
        dfs(i, odd);
        
        //每个联通块的奇点如果大于2那么应该给做过联通块添加边消去奇点
        cnt += odd > 2 ? (odd-2)/2:0; 
        cnt++;
    }
    printf("%d\n", cnt-1);
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {

        scanf("%d %d", &n, &m);
        memset(vis, 0, sizeof(vis));
        memset(book, 0, sizeof(book));
        for (int i = 0; i <= n; i++) {
            edge[i].clear();
        }

        for (int i = 0; i < m; i++) {
            int u, v;
            scanf("%d %d", &u, &v);
            book[u] = book[v] = 1;
            edge[u].push_back(v);
            edge[v].push_back(u);
        }
        solve();
    }
}

```