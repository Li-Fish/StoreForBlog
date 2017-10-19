---
title: UVA 10603 - Fill（dijkstra + 状态图）
date: 2017-02-18 16:28:33
categories: [ACM, 图论, 最短路]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1544

# 题目大意
------------------
有三个水杯，容量分别为 a, b, c 刚开始 c 水杯注满水，其他是空的，然后求经过 n 次操作后可不可以得到 d 升水。如果可以的话，转移的水量尽量少，如果无法得到 d 升的话，就输出 d' 升，**d'< d** 并且 **d'** 尽量的大。

这里的转移水量指，总共转移了多少升水，比如 a 向 b 注了五升水，那么转移水量为五升。


# 解题过程
--------------
照着紫书示例敲得，看作者代码学到了好多东西。

# 题目分析
------------------
+ 首先是进行枚举操作的时候，虽然总共三个杯子，作者还是把数据放到了数组里面，简化不少代码，如果是我的话，就直接写9个if了。

+ 然后是代码的思想是 dijkstra，求最短路。
	+ 把整个问题当做一个状态图，每个点由三个水杯里面的水确定。
	+ 两个点之间的边的权值，是由两个状态转化所需要转移的水量。（注意一点是这里是有向图）
	+ 另外三个水杯里面的水合不变，给定两个水杯里水的体积，就可以确定另一个，所以储存状态时，只需要记录两个水杯的体积即可。
+ 经过上面的解析，这个问题就抽象成了，求一个有向图的最短路径，然后使用了优先队列优化的 dijkstra 。

+ 另外memcpy的使用值得一学，使用方法如下：
`memset(&a, &b, sizeof(a));`
把 b 复制给a，直接复制的内存，效率比直接循环快。

# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 200+10;

struct Node
{
    int v[3], dist;
};

bool operator < (const struct Node& a, const struct Node& b)
{
    return a.dist > b.dist;
}

int vis[MAX][MAX], ans[MAX];

void update_ans(Node u)
{
    for (int i = 0; i < 3; i++)
    {
        int d = u.v[i];
        if (ans[d] < 0 || ans[d] > u.dist)
            ans[d] = u.dist;
    }
}

void solve(int a, int b, int c, int d)
{
    int cap[3] = {a,b,c};
    memset(vis, 0, sizeof(vis));
    memset(ans, -1, sizeof(ans));
    Node start;
    start.v[0] = 0, start.v[1] = 0, start.v[2] = c;
    start.dist = 0;
    priority_queue<Node> q;
    q.push(start);
    vis[0][0] = 1;

    while (!q.empty())
    {
        Node u = q.top(); q.pop();
        update_ans(u);

        if (ans[d] >= 0)
            break;

        for (int i = 0; i < 3; i++)
        {
            for (int j = 0; j < 3; j++)
            {
                if (i == j)
                    continue;
                if (u.v[i] == 0 || u.v[j] == cap[j])
                    continue;

                int amount = min(cap[j], u.v[i] + u.v[j]) - u.v[j];
                Node u2;
                memcpy(&u2, &u, sizeof(u));
                u2.dist = u.dist + amount;
                u2.v[i] -= amount;
                u2.v[j] += amount;
                if (!vis[u2.v[1]][u2.v[0]])
                {
                    vis[u2.v[1]][u2.v[0]] = 1;
                    q.push(u2);
                }
            }
        }
    }
}

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        int a, b, c, d;
        scanf("%d %d %d %d", &a, &b, &c, &d);
        solve(a, b, c, d);
        while (d >= 0)
        {
            if (ans[d] >= 0)
            {
                printf("%d %d\n", ans[d], d);
                break;
            }
            d--;
        }
    }
}
```
