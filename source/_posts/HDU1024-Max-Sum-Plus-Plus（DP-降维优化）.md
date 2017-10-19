---
title: HDU1024 - Max Sum Plus Plus（DP+降维优化）
date: 2017-06-30 15:18:11
categories: [ACM, DP]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=1024

------------------
# 题目大意：
给定一个长度为n的序列，一个数m，求m段不相交的区间和的最大值。

-----------------------
# 解题过程：
自己好菜啊，简单的状态转移方程都没推出来，值得以后注意的是，以后定义状态不要太”松“了。比如刚开始定义的状态$dp[i][j]$前$i$个数构成的$j$个区间和的最大值，然后发现不会转移。最后看了博客才发现别人不光是前$i$个数还要以$i$结尾，这样就转移方程就任意写出来了，虽说转移操作的复杂度增加了不少。


-----------------
# 题目分析：
首先定义状态$dp[i][j]$含义是前$i$个数以第$i$个数结尾分了$j$个区间的最大和。
那么有两种转移方式，一是把第$i$个数加到第$i-1$个数所在的区间里面，二是第$i$个数单独为一个区间，那么状态转移方程为。

$$dp[i][j] = max(dp[i-1][j], dp[k][j-1],\quad 0 < k < i,\; i \le j$$

然后这个转移方程，一个直观的写法是这样的：
```
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= i; j++) {
            dp[i][j] = dp[i-1][j];
            for (int k = 1; k < i; k++) {
                dp[i][j] = max(dp[i][j], dp[k][j-1]);
            }
        }
    }
```
显然这样时间上会超时，复杂度高达$O(n^2m)$。

这时候需要观察下上面写的代码，发现可以把n和m的循环交换顺序。

```
    for (int j = 1; j <= m; j++) {
        for (int i = j; i <= n; i++) {
            dp[i][j] = dp[i-1][j];
            for (int k = 1; k < i; k++) {
                dp[i][j] = max(dp[i][j], dp[k][j-1]);
            }
        }
    }
```

然后发现，最内层的$k$次循环其实是没有必要的，因为对于每一趟内$i$循环，都可以在上一趟循环中预处理出来最大的$k$。

```
        for (int j = 1; j <= m; j++) {
            maxn = -INF;
            for (int i = j; i <= n; i++) {
                dp[i] = max(dp[i-1], pre[i-1]) + data[i];
                pre[i-1] = maxn;
                maxn = max(maxn, dp[i]);
            }
        }
```
这里$pre$数组的含义是，$pre[i]$为从$1$到i中，最大的$dp[k][j-1]$。

---------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1123456;
const int INF = 0x7fffffff;

int data[MAX], dp[MAX], pre[MAX];

int main() {
    int m, n;
    while (~scanf("%d %d", &m, &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", data + i);
            dp[i] = pre[i] = 0;
        }
        int maxn;
        for (int j = 1; j <= m; j++) {
            maxn = -INF;
            for (int i = j; i <= n; i++) {
                dp[i] = max(dp[i-1], pre[i-1]) + data[i];
                pre[i-1] = maxn;
                maxn = max(maxn, dp[i]);
            }
        }
        printf("%d\n", maxn);
    }

    for (int j = 1; j <= m; j++) {
        for (int i = 1; i <= n; i++) {
            dp[i][j] = dp[i-1][j];
            for (int k = 1; k < i; k++) {
                dp[i][j] = max(dp[i][j], dp[k][j-1]);
            }
        }
    }
}

```
