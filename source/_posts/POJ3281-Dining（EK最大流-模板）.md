---
title: POJ3281 - Dining（EK最大流+模板）
date: 2017-07-26 10:40:33
categories: [ACM, 图论, 网络流]
tags:
---
# 题目链接：
http://poj.org/problem?id=3281

--------------------
# 题目大意：
有$N$个牛，$D$种饮料，$F$种食物，每种牛有$D_i$种想喝的饮料，$F_i$种想吃的食物，每种饮料和食物只能分配给一只牛，问最大能使多少头牛满足同时得到喜欢的饮料和喜欢的食物。

-------------------------------
# 解题过程：

还是挺好想的，一下子就想到了，然后2000长度1A美滋滋。

-------------------------------
# 题目分析：

先把每头牛拆成两个点，一个入点，一个出点，连一个容量为1的边。然后在把每头牛想要的食物与他的入点连上，每头牛想要的饮料和他的出点连上。

再建一个原点和终点，原点连上所有的食物，终点连上所有的饮料，最后跑一个最大流就可以了。上述所有的连边容量都是1。

这样建图保证，对于每一个增广路的流量都为$1$，并且都经过一头牛，一个饮料，一个食物，并且每头牛只得到过一次饮料和食物，保证不会重复计算。


----------------------
# AC代码：
```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
using namespace std;

const int MAX = 112;
const int INF = 0x3f3f3f3f;

int N, F, D;

struct Info {
    int to, rev, cap;
    Info(int to, int rev, int cap):to(to), rev(rev), cap(cap){}
};

vector<Info> edge[MAX<<2];

void add_edge(int u, int v) {
    edge[u].push_back(Info(v, edge[v].size(), 1));
    edge[v].push_back(Info(u, edge[u].size()-1, 0));
}

int prevv[MAX<<2]; //存每个节点的前驱节点
int preve[MAX<<2]; //当前节点是前驱节点的第几条边连接的
int flow[MAX<<2]; //当前节点的流量

int bfs(int src, int des) {
    queue<int> q;
    q.push(src);
    memset(prevv, -1, sizeof(prevv));
    memset(preve, -1, sizeof(preve));
    memset(flow, 0, sizeof(flow));
    prevv[src] = 0;
    flow[src] = INF; //初始化起点流量为正无穷
    while (!q.empty()) {
        int u = q.front(); q.pop();
        if (u == des) break;
        for (int i =0; i < edge[u].size(); i++) {
            Info & e = edge[u][i];
            //如果有前驱节点，说明已经走过了，就不加入队列了
            if (e.cap > 0 && prevv[e.to] == -1) {
                prevv[e.to] = u;
                preve[e.to] = i;
                //限制流量
                flow[e.to] = min(e.cap, flow[u]);
                q.push(e.to);
            }
        }
    }
    if (prevv[des] == -1) return -1;
    else return flow[des];
}

int max_flow(int src, int des) {
    int sumflow = 0, aug;
    while ((aug =bfs(src, des)) != -1) {
        int k = des;
        //遍历增广的路径
        while (k != src) {
            int last = prevv[k];
            //让路径上的边的容量减少，反向边流量增加
            edge[last][preve[k]].cap -= aug;
            edge[k][edge[last][preve[k]].rev].cap += aug;
            k = last;
        }
        sumflow += aug;
    }
    return sumflow;
}

int main() {
    while (~scanf("%d %d %d", &N, &F, &D)) {
        for (int i = 0; i <= N + F + D + 1; i++) edge[i].clear();
        for (int i = 1; i <= N; i++) {
            int f, d, v;
            scanf("%d %d", &f, &d);
            for (int j = 1; j <= f; j++) {
                scanf("%d", &v);
                v += N*2;
                add_edge(v, i);
            }
            for (int j = 1; j <= d; j++) {
                scanf("%d", &v);
                v += N*2 + F;
                add_edge(i+N, v);
            }
        }
        for (int i = 1; i <= N; i++) {
            add_edge(i, i+N);
        }
        for (int i = 1; i <= F; i++) {
            int v = i + N*2;
            add_edge(0, v);
        }
        for (int i = 1; i <= D; i++) {
            int v = i + N*2 + F;
            add_edge(v, N*2+F+D+1);
        }
        printf("%d\n", max_flow(0, N*2+F+D+1));
    }
}
```