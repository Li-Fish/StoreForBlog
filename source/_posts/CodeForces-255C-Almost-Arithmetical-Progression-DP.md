---
title: CodeForces 255C. Almost Arithmetical Progression (DP)
date: 2016-12-05 17:50:28
categories: [ACM, DP]
tags:
---
# 题目链接：

[CodeForces255C.](http://codeforces.com/contest/255/problem/C)


----------
# 题目大意：
看起来题目给的公式很复杂，其实就是找最长的 1,2,1,2 类似这样的最长子序列.
数据小于4000.


----------
# 解题过程：

 - 看到这个题首先就想到了用DP来做，毕竟正在刷DP的专题，刚开始想着这个题类似最长公共子序列那样，然后想了一个多小时也没结果，最后比赛快结束的半小时想起来这个和最长上升子序列有点像（后来发现也不是）。
 
 - 比赛完后想到了一个状态转移方程（错误的）用两个一维数组，一个用来记录以每一个数为最后一个数的最大长度，另一个储存最大长度的情况下的上一个数。
错误的状态转移方程：				 ***a[i] = a[j] + 1 (a[i] == b[i])***
  显然是错误的，于是我还考虑了下最长有多种情况的情况，用set储存上一个数，还是错误（毕竟想法就不太对）。
  
 - 于是隔了一天还是没想起来，于是百度了下题解，找到了一个不错的博客：[题解](http://blog.csdn.net/qq_24451605/article/details/48659235)
 可以看出来这篇博客风格也照抄了一下233，分析的很清楚，然后自己拿纸模拟了一遍，感觉这么简单的题怎么没想出来……


----------
# 分析过程：

 - 用a[i][j]二维数组储存状态，i代表以第几个数结束，j代表倒数第二个数。
 - 状态转移方程：***dp[i][j]  =  dp[j][k] + 1 (a[k]==a[i])***
 - 看巨巨的题解K在状态转移的过程中就可以找到了。

----------
# AC代码：

```cpp
#include <iostream>
#define MAX 4123
using namespace std;

int data[MAX];
int dp[MAX][MAX];

int main()
{
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++)
    {
        cin >> data[i];
    }

    int ans = 1;
    for (int i = 1; i <= n; i++)
    {
        int k = -1;
        for (int j = 1; j < i; j++)
        {
            if (k == -1)
                dp[i][j] = 2;
            else
                dp[i][j] = dp[j][k] + 1;
            if (data[i] == data[j])
                k = j;
            if (ans < dp[i][j])
                ans = dp[i][j];
        }
    }
    cout << ans;
}
```
# 暴力遍历代码：
看题解之前想碰碰运气写的暴力代码，时间复杂度O(n^3)，当然TLE啦……

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <set>
using namespace std;

set<int> check;
vector<int> store;
int data[41234];
int n;

int scan(int a, int b)
{
    int judge = 0;
    int sum = 0;
    for (int i = 0; i < n; i++)
    {
        if (judge == 0 && (data[i] == a || data[i] == b) || data[i] == judge)
        {
            if (data[i] == a)
            {
                judge = b;
            }
            if (data[i] == b)
            {
                judge = a;
            }
            sum++;
        }
    }
    return sum;
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i++)
    {
        cin >> data[i];
        if (check.count(data[i]) == 0)
        {
            check.insert(data[i]);
            store.push_back(data[i]);
        }
    }
    int ans = 0;
    for (int i = 0; i < store.size(); i++)
    {
        for (int j = i; j < store.size(); j++)
        {
            int t = scan(data[i], data[j]);
            ans = ans > t? ans:t;
        }
    }
    cout << ans;
}
```