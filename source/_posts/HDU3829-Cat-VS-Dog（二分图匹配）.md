---
title: HDU3829 - Cat VS Dog（二分图匹配）
date: 2017-08-18 11:30:36
categories: [ACM, 图论, 匹配]
tags:
---
# 题目链接：

http://acm.hdu.edu.cn/showproblem.php?pid=3829



--------------------
# 题目大意：

有 N 条狗， M 只猫和 P 个人。

每个人会喜欢一条狗讨厌一只猫或喜欢一只猫讨厌一条狗。

现在可以将一些狗或猫移除。如果一个人喜欢的动物被保留，并且讨厌的动物被移除，那么称这个人是开心的。现在求最多可以使多少人开心。



-------------------
# 解题过程：

刚开始一直想的是狗和猫之间建图，然后怎么想都不好解决，因为狗和猫没有直接的关系...

然后这个题放了挺久的，直到后来才想起来补题，不错的题，感觉匹配问题主要是要将原问题如何转化到图论模型上。



--------------------
# 题目分析：

首先我们对人建图。如果一个人喜欢的动物是另一个人讨厌的，那么这两个人是矛盾的，对所有矛盾的人之间建一条边。

这时候会发现，如果这样建图的话，不会出现长度为奇数的环，因为一个人只能如果喜欢猫了不能再讨厌猫。

所以这样建的图是一个二分图。对于矛盾的人，他们是不可能同时高兴的，也就是我们要尽量多的找出来一堆的人，他们之间不能有边。然后这就是最大独立集了。

对于二分图: 

+ 最大匹配 = 最小点覆盖
+ 最小点覆盖 + 最大独立集 = |V|

那么跑一边二分图最大匹配就出来答案了。



----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 2000 + 10;

struct Node {
    char like[112], dislike[112];
} data[MAX];

struct Edge {
    int u, v, nxt;
} edge[MAX * MAX * 10];

int head[MAX], etot;
int n, m, p;
int matching[MAX], vis[MAX];

void add_edge(int u, int v) {
    edge[etot].u = u;
    edge[etot].v = v;
    edge[etot].nxt = head[u];
    head[u] = etot++;
}

bool find(int u) {
    for (int i = head[u]; ~i; i = edge[i].nxt) {
        int v = edge[i].v;
        if (vis[v]) continue;
        vis[v] = true;
        if (matching[v] == -1 || find(matching[v])) {
            matching[v] = u;
            matching[u] = v;
            return true;
        }
    }
    return false;
}

int main() {
    while (~scanf("%d %d %d", &n, &m, &p)) {
        memset(head, -1, sizeof(head));
        etot = 0;
        for (int i = 0; i < p; i++) {
            scanf("%s %s", data[i].like, data[i].dislike);
        }

        for (int i = 0; i < p; i++) {
            for (int j = i + 1; j < p; j++) {
                if (strcmp(data[i].like, data[j].dislike) == 0 ||
                    strcmp(data[i].dislike, data[j].like) == 0) {
                    add_edge(i, j);
                    add_edge(j, i);
                }
            }
        }

        int ans = 0;

        memset(matching, -1, sizeof(matching));
        for (int i = 0; i < p; i++) {
            if (matching[i] != -1) continue;
            memset(vis, 0, sizeof(vis));
            if (find(i)) ans++;
        }

        printf("%d\n", p - ans);
    }
}
```