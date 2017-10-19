---
title: URAL1260 - Nudnik Photographer（DP+递推）
date: 2017-10-11 12:48:43
categories: [ACM, DP]
tags:
---
# 题目链接：

[http://acm.timus.ru/problem.aspx?space=1&num=1260](http://acm.timus.ru/problem.aspx?space=1&num=1260)

# 题目大意：

给出 1 ～ N 的 N 个数，进行排列，要求 1 一定要为第一个元素，并且任意两个相邻的元素之间的差不能超过 2，输出有多少种这样的排列。

$1\le N \le 55$

# 解题过程：

感觉是和程设课上的递推题差不多，然后卡了我了两个小时也没想出来...看来数据结构刷多了真的能刷傻。

# 题目分析：

$dp[i]$的含义是，i 个数有多少个符合上述要求排列。那么状态转移方程是：

$$dp[i] = dp[i - 3] + dp[i - 1] + 1$$

这样分情况考虑：

首先第一个位置肯定是要放 1 的（因为题目要求），然后

+ 第二个位置放 2，那么现在的排列的个数等价于 $dp[i - 1]$，你可以当做把 1 扔掉，然后剩下的数都减一
+ 第二个位置放 3
  + 第三个位置放 2，那么第四个位置一定是要放 4，那么当前情况的排列总数是等价于$dp[i - 3]$的
  + 如果第三个位置不放 2，那么只有一种情况，假设 n = 8， 那么是1 3 5 7 8 6 4 2


# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

ll dp[60];

int main() {
    int n;
    dp[1] = 1;
    dp[2] = 1;
    dp[3] = 2;
    for (int i = 4; i <= 55; i++) {
        dp[i] = dp[i - 1] + dp[i - 3] + 1;
    }
    while (~scanf("%d", &n)) {
        printf("%lld\n", dp[n]);
    }
}
```