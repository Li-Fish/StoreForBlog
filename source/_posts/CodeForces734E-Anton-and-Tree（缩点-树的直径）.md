---
title: CodeForces734E - Anton and Tree（缩点+树的直径）
date: 2017-06-27 10:17:47
categories: [ACM, 搜索, BFS]
tags:
---
# 题目链接：
http://codeforces.com/problemset/problem/734/E

------------------
# 题目大意：
给定一棵无根数，每个节点有黑白两种颜色，现在有一种涂色操作，一次可以将一块连通的同色区域涂为一个颜色，现在求最少操作几次可以将整棵树涂成黑色或白色。


-------------------------
# 解题过程：
因为之前做过一个涂板子的题，进行的操作和这个一样，也是先缩点。不过那个数据范围比较小，枚举所有点BFS一次，求最长路即可，但是做过数据范围比较大，所以用到了树的某些性质。

------------------------------
# 题目分析：

关于树的直径的介绍和BFS的求法，建议看这篇[博客](http://wattlebird.github.io/2014/09/21/%E6%A0%91%E7%9A%84%E7%9B%B4%E5%BE%84/)，也有详细证明。

如果之前做过[这道题](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3781)的话，那么这个题的思路不难想，就是缩点然后再求一下树的直径就好了。

树的直径有一个性质，以树的直径中点上的一点为根，将这颗无根树转化为有根树的话，这样做会使树的深度最小，且为树的直径的一半（向上取整），因为如果有一个叶子节点比直径更深的话，那么就可以构成一个更长的直径，与假设矛盾。

另外在树上缩点，构成的新的图也是一棵树。

-------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 212345;

int color[MAX], n, vis[MAX], top;
vector<int> raw_edge[MAX], edge[MAX];

//用DFS染色缩点
void dfs(int u) {
    vis[u] = top;
    for (int i = 0; i < raw_edge[u].size(); i++) {
        int v = raw_edge[u][i];
        if (!vis[v] && color[u] == color[v]) {
            //如果颜色相同，并且没访问过那么缩成一个点
            dfs(v);
        }
        else if (vis[v] && color[u] != color[v]){
            //如果颜色不同，并且访问过，那么就建边
            edge[top].push_back(vis[v]);
            edge[vis[v]].push_back(top);
        }
    }
}

void build() {
    top = 1;
    memset(vis, 0, sizeof(vis));
    for (int i = 1; i <= n; i++) {
        if (!vis[i]) {
            dfs(i);
            top++;
        }
    }
}

//BFS求最长路
int bfs(int u) {
    memset(vis, 0, sizeof(vis));
    queue<int> q;
    q.push(u);
    vis[u] = 1;
    while (!q.empty()) {
        u = q.front();
        q.pop();
        for (int i = 0; i < edge[u].size(); i++) {
            int v = edge[u][i];
            if (!vis[v]) {
                vis[v] = vis[u] + 1;
                q.push(v);
            }
        }
    }
    int pos = 1;
    for (int i = 1; i < top; i++) {
        if (vis[i] > vis[pos])
            pos = i;
    }
    return pos;
}

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> color[i];
    }
    for (int i = 0; i < n-1; i++) {
        int u, v;
        scanf("%d %d", &u, &v);
        raw_edge[u].push_back(v);
        raw_edge[v].push_back(u);
    }
    build();
    //先确定直径的一个端点
    int x = bfs(1);
    //然后从端点跑最长路求出直径长度，这里距离求出来的是比实际距离大1，所以默认向上取整
    printf("%d\n", vis[bfs(x)]/2);
}
```
