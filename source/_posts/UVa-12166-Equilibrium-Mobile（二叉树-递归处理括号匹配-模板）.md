---
title: UVa 12166 - Equilibrium Mobile（二叉树+递归处理括号匹配+模板）
date: 2017-02-08 17:37:38
categories: [ACM, 数据结构]
tags:
---
# 题目链接
[题目](https://vjudge.net/problem/24840/origin)

Mark下大神的博客：http://morris821028.github.io/
简直太强了。
题解链接：http://morris821028.github.io/2014/10/03/oj/uva/uva-12166/#Problem

----------
# 题目大意：
給一個天平表達式，請問至少要調整幾個權重才能使之平衡。（直接复制来的）


----------
# 解题过程：
自己大概废了一个小时想一个特麻烦的解法，首先想的是自顶向下的平衡，然后dfs下去还是从必须从叶节点开始平衡。

于是想自底向上平衡，每次把可以平衡成的质量和调整的次数传给上一层，比如调整[1,2]，给上一层传递三个状态：调整2到1，质量变为1，调整1次；调整1变为2，质量变为2，调整一次；两个都一起调整到一个任意的数，调整两次。
显然这样需要给每个节点开辟一个空间储存状态，妥妥爆内存。

想了一个小时只是这个结果，于是去百度了下，看到：
**那麼可以得知道假使一個權重不改，最後的天平重量為何。**
**假使 depth 上的權重 w 不改，則最後的天平重量就是 w * pow(2, depth)。**

于是想到建树，统计下叶子节点所在的层数，然后拿每个叶子节点跑一边，结果是O(n^2)。

后来看到这个博客确实是惊艳到了……


----------
# 题目分析：
这里分析下下面的代码好了。

这里map的使用和遍历可以做模板了，要是我的话，就桶排然后遍历一遍了...

这个递归写的真是太强了！！

然后就是sscanf的用法，可以拿来做模板，要我的话，专门写一个字符串转整数的函数了，麻烦的要死。

----------
# AC代码：

```cpp
#include <cstdio>
#include <cstring>
#include <map>
using namespace std;

char exp[1123456];
map<long long, int> R;

void dfs(int l, int r, int dep)
{
    if (exp[l] == '[')
    {
        int p = 0;
        for (int i = l + 1; i <= r - 1; i++)
        {
            if (exp[i] == '[')
                p++;
            if (exp[i] == ']')
                p--;
            if (p == 0 && exp[i] == ',')
            {
                dfs(l+1, i-1, dep+1);
                dfs(i+1, r-1, dep+1);
            }
        }
    }
    else
    {
        int w;
        exp[r+1] = '\0';
        sscanf(exp+l, "%d", &w);
        R[(long long)w<<dep]++;
    }
}

int main()
{
    int testcase;
    scanf("%d", &testcase);
    while (testcase--)
    {
        scanf("%s", exp);
        R.clear();
        dfs(0,strlen(exp) - 1, 0);
        int sum = 0, mx = 0;
        for (map<long long, int>::iterator it = R.begin(); it != R.end(); it++)
            sum += it->second, mx = max(mx, it->second);
        printf("%d\n", sum - mx);
    }
    return 0;
}

```