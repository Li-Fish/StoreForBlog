---
title: 最长上升子序列（DP+模板）
date: 2017-04-09 16:22:35
categories: [ACM, DP]
tags:
---
# 题目链接：
 http://poj.org/problem?id=1631
 
-------------------------------------
# 题目大意：
 有两个不可描述的线段，每个上面有 n 个接口，现在给定了一个连接，求如果减去一些连接的话，最大的不交叉连接个数是多少。

-----------------------------------------------
# 解题过程：
 省赛选拔赛的题，英文题面太长直接没看。
 理解题意后挺简单的，只要找到规律。

---------------
# 题目分析：
  要求最大的不交叉，可以找到一个规律，就是求不递减子序列，不过这里用 O(n^2) 的会超时，所以用了一个 O(nlongn) 的模板。

# AC代码：
```cpp
#include<cstdio>
#include<cstring>
#include<algorithm>
using namespace std;

int dp[40000+100];

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, ans = 0, t;
        scanf("%d", &n);
        for (int i = 0; i < n; i++) {
            scanf("%d", &t);
            if (!ans || t >= dp[ans-1])
                dp[ans++] = t;
            else
                dp[lower_bound(dp, dp+ans, t)-dp] = t;
        }
        printf("%d\n", ans);
    }
}
```