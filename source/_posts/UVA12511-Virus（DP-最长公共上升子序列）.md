---
title: UVA12511 - Virus（DP+最长公共上升子序列）
date: 2017-06-11 21:22:11
categories: [ACM, DP]
tags:
---
# 题目链接：
https://vjudge.net/problem/UVA-12511

----------------------------------
# 题目大意：
给定两个序列，求出两个序列的最长公共上升子序列（严格上升）。


-----------------------------------------
# 解题过程：
比赛的时候没有做出来，非常咸鱼的一场比赛，当时是想错了状态。当时想的状态是定义$dp[i][j]$，意味以第一个串第前i个元素，第二个串前j个元素的最长公共上升子序列长度。

但是这样定义状态有后效性，比如当前我知道$dp[i][j]$要以这个状态进行转移的话，需要他是以那个状态转移而来的，换句话说，我转移的时候要知道他是以前j个数中那一个结尾的。

如果换一种方式，$dp[i][j]$代表以第一个序列前i个元素并且以第i个结束，第二个序列前j个元素并且以第j个元素结尾的最长上升子序列的长度。

这样加入的限制太多，不容易找出状态转移方程，或者转移起来太麻烦。

--------------------------
# 题目分析：
这里以$dp[i][j]$表示第一个序列中前i个元素，第二个序列前j个元素并且以第j个元素为结尾的最长上升子序列。

这样对比前两种状态表示方式有两种好处，一是无后效性，$dp[i][j]$的第二维就确定了这个序列是以那一个元素结尾。二是容易进行转移，对于$dp[i][j]$可由两种方式转移而来：


$$
dp[i][j] = 	
\begin{cases}
dp[i-1][j] , &a[i]  \ne b[i] \\
max(dp[i-1][k])+1, &k \in [1, j-1] \wedge b[k] < b[j] \wedge a[i] = b[i]
\end{cases}
$$

这里的k可以在循环中找出，时间复杂度为$O(n^2)$.

-----------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1123;

int dp[MAX][MAX], a[MAX], b[MAX];

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, m;
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
        }
        scanf("%d", &m);
        for (int i = 1; i <= m; i++) {
            scanf("%d", &b[i]);
        }
        memset(dp, 0, sizeof(dp));
        for (int i = 1; i <= n; i++) {
            int maxn = 0;
            for (int j = 1; j <= m; j++) {
                //不相等时的转移
                dp[i][j] = dp[i-1][j];
                //更新maxn变量，表示当前小于a[i]的dp[i-1][k]的最大值
                if (a[i] > b[j] && maxn < dp[i-1][j])
                    maxn = dp[i-1][j];
                //相等的话
                if (a[i] == b[j])
                    dp[i][j] = maxn+1;
            }
        }
        int ans = 0;
        for (int i = 1; i <= m; i++) {
            ans = max(ans, dp[n][i]);
        }
        printf("%d\n", ans);
    }
}
```


