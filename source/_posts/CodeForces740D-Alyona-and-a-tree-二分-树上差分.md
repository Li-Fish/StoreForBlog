---
title: CodeForces740D - Alyona and a tree (二分+树上差分)
date: 2017-08-16 22:11:26
categories: [ACM, 二分]
tags:
---
# 题目链接：

http://codeforces.com/problemset/problem/740/D



--------------------
# 题目大意：

给出一颗有根树，每个点有点权$a_i$，每个边有边权，定义$dist(u, v)$为 u 到 v 的边权和，如果有两个节点符合，v 是 u 的子节点并且 $dist(u, v) \le a_v$那么称 v 为 u 控制的节点。

题目要求输出每个点控制的节点有多少个。



-------------------
# 解题过程：

大体思路没错，推出来如果固定 v 的话，那么从 v 往上会有一段连续的祖先都可以控制 v，让他们控制的节点都加一就可以了，然后找这一段连续的祖先可以用二分去找。但是不知道该怎么让这一对祖先同时加一，于是想起来之前刚学的树链剖分，感觉 CF div2 的 D 题不至于上树剖...

然后去翻了下博客，发现了居然还有树上差分这种东西！问了学长 + 翻了 standing 上的代码算是理解了。



--------------------
# 题目分析：

首先我们设 $dep[u] $ 为根节点到节点 u 的权值和，对于原式 $dist(u, v) \le a_v$ 可化为 $dep[v] - a_v \le dep[u]$ ，这样我们在递归到 v 的时候，维护一个序列以深度的顺序记录 v 的所有祖先的 $dep$ 值，然后这个值肯定是递增的。

这样对于每个节点 v 用二分在序列里找到满足上式最小的$dep[u]$，那么序列中 u 后面的节点都满足上式，让这些节点控制的节点数全加一即可。

对于加一这个操作，我们让第一个祖先节点的父亲减一，当前节点的父亲加一，然后你会会发现，现在每个节点的所有子节点和他自身的和就是答案了。







----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long LL;

const int MAX = 212345;

int n, idx[MAX], pcnt;
//pre记录当前节点所有父亲节点的dep，并且是递增的
//idx记录pre数组里每个位置对应的下标
//d记录当前节点+1/-1操作了多少次
//ans记录答案
LL a[MAX], w[MAX], pre[MAX], d[MAX], ans[MAX];

vector<int> G[MAX];

void dfs(int u) {
	//找出第一个符合的祖先
    int id = lower_bound(pre, pre + pcnt + 1, pre[pcnt] - a[u]) - pre;
    //进行+1/-1操作
    d[idx[id - 1]]--;
    d[idx[pcnt - 1]]++;
    for (int i = 0; i < G[u].size(); i++) {
        int v = G[u][i];
        //更新pre数组
        pre[pcnt + 1] = pre[pcnt] + w[v];
        idx[pcnt + 1] = v;
        pcnt++;
        dfs(v);
        pcnt--;
        //累加每个儿子的权值和
        ans[u] += ans[v];
    }
    ans[u] += d[u];
}

int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);
    for (int v = 2; v <= n; v++) {
        int u;
        scanf("%d%lld", &u, &w[v]);
        G[u].push_back(v);
    }
    idx[0] = 1;
    dfs(1);
    for (int i = 1; i <= n; i++) printf("%lld ", ans[i]);
}
```