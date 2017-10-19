---
title: ZOJ3781 - Paint the Grid Reloaded（缩点+最短路）
date: 2017-06-13 11:36:05
categories: [ACM, 搜索, BFS]
tags:
---
# 题目链接：
http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3781

--------------------------
# 题目大意：
给定一个$N\times M$的矩阵，每个格子涂着黑色或白色。现在有一种涂色操作，每次涂色可以将一个格子与这个格子连通的格子涂成一个颜色。连通是指上下左右的边相接。

求最少的操作次数，将这个矩阵涂成一种颜色。

-----------------------
# 解题过程：
很久以前比赛的题，当时看到这个题一点想法都没有，后来补题看到了是缩点和求最长路，感觉非常神奇，也是第一次接触缩点的思想。

----------------------------
# 题目分析：

因为每次操作可以将相邻的格子涂成一个颜色，那么我可以将相同颜色连通的格子缩成一个点，与这一块连通格子相邻的相反颜色的点建边。

我们以一块连通块为中心，不断重复的将这一连通块涂成相反的颜色，那么最终会把整个矩阵涂成一个颜色。因为将这个连通块涂成反色的话，这个连通块就会和周围反色的连通块连成一个更大的连通块。

那么最少的操作次数，就遍历所有的点，以某个点为起点，BFS求出的最大步数为以这个点为起点的最少操作数。

为什么这样操作是最少的呢？这样建图构成的是一个二分图，然后想一下就知道了。


# AC代码：
## BFS染色：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1600+5, INF = 0x3f3f3f3f;

int n, m, top;
char map_data[MAX][MAX];
int data[MAX][MAX], vis[MAX];
int dirc[4][2] = {{1,0},{0,1},{-1,0},{0,-1}};
vector<int> G[MAX];

//判断是否出界
inline bool edge(int x, int y) {
    return x >= 0 && y >= 0 && x < n && y < m;
}


//用来BFS给图染色
void bfs(int x, int y) {
    queue<pair<int, int> > q;
    data[x][y] = top;
    q.push(make_pair(x, y));
    while (!q.empty()) {
        x = q.front().first;
        y = q.front().second;
        q.pop();
        for (int i = 0; i < 4; i++) {
            int tx = x + dirc[i][0];
            int ty = y + dirc[i][1];
            if (edge(tx, ty) && map_data[x][y] == map_data[tx][ty] && !data[tx][ty]) {
                q.push(make_pair(tx, ty));
                data[tx][ty] = top;
            }
            else if (edge(tx, ty) && data[tx][ty] && data[tx][ty] != data[x][y]) {
                int a = data[x][y], b = data[tx][ty];
                G[a].push_back(b);
                G[b].push_back(a);
            }
        }
    }
}


//染色
void flood_fill() {
    memset(data, 0, sizeof(data));
    top = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (!data[i][j]) {
                top++;
                bfs(i, j);
            }
        }
    }
}

//找出以当前点为起点的最长路
int check_depth(int u) {
    queue<pair<int, int> > q;
    q.push(make_pair(u, 0));
    memset(vis, 0, sizeof(vis));
    vis[u] = 1;
    int max_depth = 0;
    while (!q.empty()) {
        u = q.front().first;
        int depth = q.front().second;
        max_depth = max(depth, max_depth);
        q.pop();
        for (int i = 0; i < G[u].size(); i++) {
            int v = G[u][i];
            if (!vis[v]) {
                q.push(make_pair(v, depth+1));
                vis[v] = 1;
            }
        }
    }
    return max_depth;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        scanf("%d %d", &n, &m);

        for (int i = 0; i <= n*m; i++) {
            G[i].clear();
        }

        for (int i = 0; i < n; i++) {
            scanf("%s", map_data[i]);
        }
        flood_fill();
        int ans = INF;
        for (int i = 1; i <= top; i++) {
            ans = min(check_depth(i), ans);
        }
        printf("%d\n", ans);
    }
}
```


## DFS染色：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1600+5, INF = 0x3f3f3f3f;

int n, m, top;
char map_data[MAX][MAX];
int data[MAX][MAX], vis[MAX];
int dirc[4][2] = {{1,0},{0,1},{-1,0},{0,-1}};

vector<int> G[MAX];

inline bool edge(int x, int y) {
    return x >= 0 && y >= 0 && x < n && y < m;
}

void dfs(int x, int y, int num, char c) {
    for (int i = 0; i < 4; i++) {
        int tx = x + dirc[i][0];
        int ty = y + dirc[i][1];
        if (!edge(tx, ty))
            continue;
        if (!data[tx][ty] && map_data[tx][ty] == c) {
            data[tx][ty] = num;
            dfs(tx, ty, num, c);
        }
        else if (map_data[tx][ty] != c && data[tx][ty]) {
            int v = data[tx][ty];
            G[v].push_back(num);
            G[num].push_back(v);
        }
    }
}

void flood_fill() {
    memset(data, 0, sizeof(data));
    top = 0;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (!data[i][j]) {
                data[i][j] = ++top;
                dfs(i, j, top, map_data[i][j]);
            }
        }
    }
}

int check_depth(int u) {
    queue<pair<int, int> > q;
    q.push(make_pair(u, 0));
    memset(vis, 0, sizeof(vis));
    vis[u] = 1;
    int max_depth = 0;
    while (!q.empty()) {
        u = q.front().first;
        int depth = q.front().second;
        max_depth = max(depth, max_depth);
        q.pop();
        for (int i = 0; i < G[u].size(); i++) {
            int v = G[u][i];
            if (!vis[v]) {
                q.push(make_pair(v, depth+1));
                vis[v] = 1;
            }
        }
    }
    return max_depth;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        scanf("%d %d", &n, &m);

        for (int i = 0; i <= n*m; i++) {
            G[i].clear();
        }

        for (int i = 0; i < n; i++) {
            scanf("%s", map_data[i]);
        }
        flood_fill();
        int ans = INF;
        for (int i = 1; i <= top; i++) {
            ans = min(check_depth(i), ans);
        }
        printf("%d\n", ans);
    }
}
```