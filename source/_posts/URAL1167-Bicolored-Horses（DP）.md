---
title: URAL1167 - Bicolored Horses（DP）
date: 2017-10-12 09:11:53
categories: [ACM, DP]
tags:
---
# 题目链接：

[http://acm.timus.ru/problem.aspx?space=1&num=1167](http://acm.timus.ru/problem.aspx?space=1&num=1167)



# 题目大意：

给出一段 0 和 1 的串，要求将其划分成 k 个连续的部分（一定要划分成 k 个连续的部分，并且某一部分不能为 0，任一 0 或 1 一定要分到一分组里），记某一分组有 i 个 0 和 j 个 1，那么这一组的值为 $i \cdot j$ ，现在要求划分 k 组，使得这 k 组的值和最小，输出最小值。

$1 \le N \le 500, 1\le K \le N$



# 解题过程：

先是从四维降到了三维，然后还是 MLE，最后类比了一下 M 子段和的题，还是不对，然后翻的别人博客，看到状态转移方程后感觉好像也不是太难...



# 题目分析：

设 $dp[i][j]$ 的含义是划分了前 i 组以第 j 个元素为结尾的最小值。

那么状态转移方程是:

$$dp[i][j] = min(dp[i - 1][k] + v | k = 0 \dots j - 1)$$

含义是，因为分组是连续的第 i 组以 j 结尾可以枚举第 i - 1 组以谁结尾来转移。

这里的 v 是前缀和，含义是 $[k + 1 , j]$ 这段区间的值。




# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 500 + 5;

int dp[MAX][MAX];
int data[MAX];
int one[MAX];
int zero[MAX];

int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m)) {
        memset(zero, 0, sizeof(zero));
        memset(one, 0, sizeof(one));
        for (int i = 1; i <= n; i++) {
            scanf("%d", data + i);
            if (data[i]) one[i] += 1;
            else zero[i] += 1;
            one[i] += one[i - 1];
            zero[i] += zero[i - 1];
        }
        memset(dp, 0x3f, sizeof(dp));
        dp[0][0] = 0;

        for (int i = 1; i <= m; i++) {
            for (int j = i; j <= n; j++) {
                for (int k = i - 1; k < j; k++) {
                    int v;
                    v = zero[j] - zero[k];
                    v *= one[j] - one[k];
                    dp[i][j] = min(dp[i - 1][k] + v, dp[i][j]);
                }
            }
        }
        printf("%d\n", dp[m][n]);
    }
}
```