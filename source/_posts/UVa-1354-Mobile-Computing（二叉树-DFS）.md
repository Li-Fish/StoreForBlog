---
title: UVa 1354 - Mobile Computing（二叉树 + DFS）
date: 2017-02-18 16:53:24
categories: [ACM, 数据结构]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4100

# 题目大意
------------------
给定多个二叉树的叶子，每个节点有一个重量，两个节点可以用一个木棍连接，一个木棍的一头可以连接一个叶子节点，或者另一根木棍。

要保证每个木棍平衡，符合胡克定律。

# 解题过程
--------------
刚开始写的，状态是用一个 vector 储存的，每次删去两个节点，继续归并其他的。
然后来回操作非常麻烦，也一直 WA。

于是百度了下别人的博客，他是每一层 dfs 都开一个 vector 数组储存状态，然后以引用传递，传给下级。

每层一个 vector 储存当前的状态，避免了来回操作一个数组造成 bug。

# 题目分析
------------------
主要是注意下，这里 dfs 时每层储存状态。

# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 112;
const double EPS = 1e-8;

struct node
{
    double first;
    double second;
    double right;
    node (double first, double second, double right):first(first),second(second),right(right){}
};

int n;
double width, ans = -1;
vector<node> status;

node connect(node a, node b)
{
    double w = a.second + b.second;
    double la = b.second / w, lb = a.second / w;
    node ans = node(max(a.first + la, -lb + b.first), w,max(b.right + lb, -la + a.right));
    return ans;
}

void solve(vector<node> &status)
{
    if (status.size() == 1)
    {
        double t = status[0].first + status[0].right;
        if (t <= width + EPS)
            ans = max(ans, t);
        return;
    }

    for (int i = 0; i < status.size(); i++)
    {
        for (int j = 0; j < status.size(); j++)
        {
            if (i == j)
                continue;

            vector<node> nstatus;

            for (int k = 0; k < status.size(); k++)
            {
                if (k == i || k == j)
                    continue;
                nstatus.push_back(status[k]);
            }

            node a = status[i], b = status[j];
            node x = connect(a, b);

            if (x.first + x.right > width + EPS)
                continue;

            nstatus.push_back(x);
            solve(nstatus);
        }
    }
}

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        status.clear();
        ans = -1;
        scanf("%lf", &width);
        scanf("%d", &n);
        while (n--)
        {
            double w;
            scanf("%lf", &w);
            status.push_back(node(0.0, w, 0.0));
        }
        solve(status);
        if (ans < 0)
            puts("-1");
        else
            printf("%.16lf\n", ans);
    }
}
```