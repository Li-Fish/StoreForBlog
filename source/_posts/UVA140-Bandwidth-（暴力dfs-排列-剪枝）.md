---
title: UVA140 - Bandwidth （暴力dfs+排列+剪枝）
date: 2017-02-15 21:09:14
categories: [ACM, 搜索]
tags:
---
# 题目链接：
--------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=76

# 题目大意
----------
有一个图，有n个节点 (n < 8) ，讲这些节点排成一列。定义节点i的带宽为与相邻节点在排列中的最远距离，所有节点的带宽最大值为图的带宽。求将这些节点排列后，带宽最小的一种排列方式。

# 解题过程：
---------
这个题写了两遍，之前一次，写了差不多一半了，不在状态，感觉又很烦，于是直接不想写了。今天网上状态很好，正好切换下了命名规范，以后还是以下划线分割好了，普通变量名和函数小写，类首字母大写。

这里把这道题放上来，是因为 get 了新知识，**解答树的剪枝**，当一种情况已经预知到不符合条件的时候，就不需要继续 dfs 下去了，直接舍弃，相当于剪掉了解答树的一条分支。

# 题目分析：
---------------
+ 先处理输入数据，这里自己写了两个辅助函数，用来处理输入数据和初始化一些变量。
+ 这里选择用矩阵储存图，之前用邻接表感觉太麻烦了，因为直接添加的话，可能有重边。
+ 然后是类似紫书上面全排列示例程序那样写的 dfs 函数，需要注意的是剪枝，和标记。
	+ 当一个节点还有m个相邻的节点未分配时，如果 m >= k (当前取得的最小带宽) 时就舍弃掉这种情况，因为就算后面 m 个节点和当前节点紧挨着，那么最小带宽也是 m ，题目要求是最小字典序的所以等于 k 的情况也舍掉。
	+ 另外如果一个节点的相邻节点已经分配了，如果这两个节点的距离大于等于 k 也舍弃掉，理由同上。
	+ 每次尝试后，都需要把标记撤回。
# AC代码：
------------------
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 30;
bool edges[MAX][MAX];
char raw_data[1123];
int node_num, book_num[MAX];
int pos[MAX], ans[MAX], k;

void put()
{
    char result[112];

    for (int i = 0; i < MAX; i++)
    {
        if (ans[i] != -1)
            result[ans[i]] = 'A'+i;
    }

    for (int i = 0; i < node_num; i++)
        printf("%c ", result[i]);
    printf("-> %d\n", k);
}


void add_edge(int l, int r)
{
    int u = raw_data[l] - 'A';

    for (int i = l+2; i <= r; i++)
    {
        int v = raw_data[i] - 'A';
        edges[u][v] = 1;
        edges[v][u] = 1;
    }
}

void analysis()
{
    node_num = 0, k = 9999;
    memset(pos, -1, sizeof(pos));
    memset(edges, 0, sizeof(edges));
    memset(book_num, 0, sizeof(book_num));

    int len = strlen(raw_data);

    int head = 0;
    for (int i = 0; i < len; i++)
    {
        if (isalpha(raw_data[i]) && book_num[raw_data[i]-'A'] == 0)
        {
            book_num[raw_data[i]-'A'] = 1;
            node_num++;
        }
        if (i == len-1 || raw_data[i+1] == ';')
        {
            add_edge(head, i);
            head = i+2;
        }
    }
}

void solve(int cur, int cnt)
{
    if (cur == node_num)
    {
        if (cnt < k)
        {
            for (int i = 0 ; i < MAX; i++)
                ans[i] = pos[i];
            k = cnt;
            return;
        }
    }
    else
    {
        for (int i = 0; i < MAX; i++)
        {
            if (!book_num[i] || pos[i] != -1)
                continue;

            pos[i] = cur;

            int dis = 0, flag = 1;
            for (int j = 0; j < MAX; j++)
            {
                if (edges[i][j])
                {
                    if (pos[j] == -1)
                        dis++;
                    else if (cur - pos[j] >= k)
                    {
                        flag = 0;
                        break;
                    }
                    else
                        cnt = max(cnt, cur - pos[j]);
                }
            }

            if (dis >= k)
                flag = 0;

            if (flag)
                solve(cur+1, cnt);

            pos[i] = -1;
        }
    }
}



int main()
{
    while (cin >> raw_data && raw_data[0] != '#')
    {
        analysis();
        solve(0, 0);
        put();
    }
}

```

# 小结：
-------------------
感觉有时候状态真的挺重要的，没状态的时候，写的很长，而且很乱。有状态的时候，写的很长，但是写的很爽，各种函数，功能分离开来，单独调试。这题看着很麻烦，写起来也很麻烦，但是这次状态很好，然后写起来顺心的话，细节也不容易出bug，写出来过了样例就一次AC了。感觉好玄学的样子。