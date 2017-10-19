---
title: SDUT 2622 - 最短路径（SPFA+二维）
date: 2017-02-16 15:14:13
categories: [ACM, 图论, 最短路]
tags:
---
# 题目链接
-------------------
http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/2622.html

# 题目大意
------------------
有一个有向图，给定一个终点和起点，求起点到终点的最短路径，并且路径经过的边数是 x 的倍数。

# 解题过程
--------------
想了好长时间，最初是想把每次的步数一起装到队列里面，用 SPFA 。
然后 WA ， 只好去搜了下博客，原来是多了个维度，和之前做的一个题神似，[UVA1600](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4475) ，这个是多了一个维度的BFS，以后得想起来这个加一个维度解决问题的方法了，要不遇到就卡死。

# 题目分析
------------------
+ 大体思路是用 SPFA ，增加一个维度，储存步数对 x 取模，最后输出对 x 取模后是 0 的终点距离即可。

# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

struct node
{
    int v;
    long long w;
    node(int v, long long w):v(v),w(w){}
};

vector<node> edges[112];
bool book[112];
long long dis[112][11];
int s, e, x;
int n, m;

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d %d", &n, &m);
        memset(book, 0, sizeof(book));
        memset(dis, -1, sizeof(dis));
        for (int i = 0; i < n; i++)
            edges[i].clear();

        for (int i = 0; i < m; i++)
        {
            int u, v;
            long long w;
            scanf("%d %d %lld", &u, &v, &w);
            edges[u].push_back(node(v,w));
        }

        scanf("%d %d %d", &s, &e, &x);
        queue<int> q;
        q.push(s);
        dis[s][0] = 0;

        while (!q.empty())
        {
            int u = q.front();

            for (int i = 0; i < edges[u].size(); i++)
            {
                int v = edges[u][i].v;
                long long w = edges[u][i].w;
                for (int k = 0; k < x; k++)
                {
                    if (dis[u][k] != -1 && (dis[v][(k+1)%x] > dis[u][k] + w || dis[v][(k+1)%x] == -1))
                    {
                        dis[v][(k+1)%x] = dis[u][k] + w;
                        if (!book[v])
                        {
                            q.push(v);
                            book[v] = 1;
                        }
                    }
                }
            }
            book[u] = 0;
            q.pop();
        }

        if (dis[e][0] == -1)
            printf("No Answer!\n");
        else
            printf("%lld\n", dis[e][0]);
    }
}
```