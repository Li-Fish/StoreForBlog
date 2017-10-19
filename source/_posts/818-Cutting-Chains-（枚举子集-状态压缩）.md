---
title: 818 - Cutting Chains （枚举子集 + 状态压缩）
date: 2017-03-14 19:13:13
categories: [ACM, 搜索]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=759
# 题目大意
------------------
给定n个环，其中有些环可以扣在一起，一个环可以和多个环扣在一起。
现在需要求最少打开多少个环才能使这些环构成一条链。（当然打开了环还需要扣上，打开扣上算一次操作）
# 解题过程
--------------
 既然是暴力里面的题，自然应该是暴力解决的。然后思路是枚举每一个环，枚举他的所有删边增边情况。大体用 IDA* 搞一搞。
然后超时了，跑到第四层就要好长时间，第五层直接走不动了。
然后就去翻了翻博客，原来是位运算枚举子集。
# 题目分析
------------------
+ 用集合表示每一个环打不打开的情况，首先所有的环至多打开一次。
+ 然后用二进制枚举枚举子集。
+ 统计下每个环的度，如果度大于 2，那么一定构不成链。
+ dfs 一下删除打开的环后，剩下的连通块有没有构成环。
+ 最后以打开的环当做边，看看能否把这些独立的链连起来。如果打开的环数大于等于独立的链的条数减一，即可连起来。


# __builtin_popcount() 函数使用
```cpp
int n;
__builtin_popcount(n);
```

函数返回 n 的二进制表示中 1 的个数。
用二进制枚举子集的时候可以用来统计 1 的个数。

# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

int g[16][16], gmask[16];
int visited[16];

int dfs(int u, int p, int open, int n) {
    visited[u] = 1;
    for (int i = 0; i < n; i++) {
        if ((open>>i)&1)
            continue;
        if (g[u][i] == 0 || i == p)
            continue;
        if (visited[i] || dfs(i, u, open, n))
            return 1;
    }
    return 0;
}

int checkChain(int open, int n) {
    for (int i = 0; i < n; i++) {
        if ((open>>i)&1)
            continue;
        int t = gmask[i]^(gmask[i]&open);
        int degree = __builtin_popcount(t);
        if (degree > 2)
            return 0;
    }

    int op = __builtin_popcount(open);
    int comp = 0;
    memset(visited, 0, sizeof(visited));
    for (int i = 0; i < n; i++) {
        if ((open>>i)&1)
            continue;
        if (visited[i])
            continue;
        if (dfs(i, -1, open, n))
            return 0;
        comp++;
    }
    return op >= comp - 1;
}

int main() {
    int n, cases = 0;
    while (~scanf("%d", &n) && n) {
        memset(g, 0, sizeof(g));
        memset(gmask, 0, sizeof(gmask));
        while (true) {
            int a, b;
            scanf("%d %d", &a, &b);
            if (a == -1 && b == -1)
                break;
            a--, b--;
            g[a][b] = g[b][a] = 1;
            gmask[a] |= 1<<b, gmask[b] |= 1<<a;
        }

        int ret = 0x3f3f3f3f;
        for (int i = 0; i < (1<<n); i++) {
            int op = __builtin_popcount(i);
            if (op >= ret)
                continue;
            if (checkChain(i, n))
                ret = min(ret, op);
        }
        printf("Set %d: Minimum links to open is %d\n", ++cases, ret);
    }
}

```