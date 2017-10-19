---
title: CodeForces743D - Chloe and pleasant prizes（树型DP）
date: 2017-06-27 17:10:50
categories: [ACM, DP, 树型DP]
tags:
---
# 题目链接：
https://vjudge.net/problem/CodeForces-743D


----------------------
# 题目大意：
给定一颗树，每个节点上有一个权值，求找出两颗不相交子树，使两颗子树的权值和最大。

---------------------------------
# 解题过程：
好久好久好久之前CF比赛的题，当时好像是没读懂题意，虽然说现在也有点读不懂，最后看了下别人的博客才知道题意。看到下面的标签上有DFS和DP，于是往树型DP上想，不过没系统的学过，之前只写过一个求树的重心的，于是照着那个思路想，没想到还真的可以做。

-------------------------------
# 题目分析：
定义状态$dp[u]$表示以节点$u$的所有子树（包括他自身）的最大权值和，$sum(u)$表示以$u$为根节点的树的权值和显然有：
$dp[u] = max(dp[v], sum(u))$  
这里的$v$是$u$的儿子节点。

然后要求不相交的两个子树，对于一个节点，他的两个儿子构成的子树一定是不相交的，这样遍历一下所有的节点就好了。


-----------------------------
# AC代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;

const int MAX = 212345;
const LL INF = 0x3f3f3f3f3f3f3f3f;

vector<int> edge[MAX];
int data[MAX];
bool vis[MAX];

int fat[MAX];
LL sum[MAX], dp[MAX];

//DFS求出DP数组
LL dfs(int u) {
    vis[u] = true;
    //sum记录以当前节点为根节点的权值和
    LL & temp = sum[u];
    temp += data[u];
    //枚举所有的儿子节点，更新DP数组和计算当前树的权值和
    for (int i = 0; i < edge[u].size(); i++) {
        int v = edge[u][i];
        if (!vis[v]) {
            fat[v] = u;
            LL t = dfs(v);
            temp += t;
            dp[u] = max(dp[v], dp[u]);
        }
    }
    //最后用当前树的权值和更新一下DP数组
    dp[u] = max(dp[u], temp);
    return temp;
}

int main() {
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        dp[i] = -INF;
        cin >> data[i];
    }
    for (int i = 0; i < n-1; i++) {
        int u, v;
        cin >> u >> v;
        edge[u].push_back(v);
        edge[v].push_back(u);
    }
    dfs(1);
    LL ans = -INF;
    for (int i = 1; i <= n; i++) {
        if (edge[i].size() < 2)
            continue;
        LL a, b;
        a = b = -INF;
        //枚举每个节点，找出最大的两个儿子节点
        for (int j = 0; j < edge[i].size(); j++) {
            int v = edge[i][j];
            if (fat[v] != i)
                continue;
            if (a < dp[v]) {
                a = dp[v];
                if (b < a)
                    swap(a, b);
            }
        }
        if (a == -INF || b == -INF)
            continue;
        ans = max(ans, a + b);
    }

    if (ans <= -INF)
        printf("Impossible\n");
    else
        printf("%lld\n", ans);
}
```