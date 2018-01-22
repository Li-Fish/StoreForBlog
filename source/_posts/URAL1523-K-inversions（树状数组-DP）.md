---
title: URAL1523 - K-inversions（树状数组+DP）
date: 2017-12-06 13:33:50
categories: [ACM, DP]
tags:
---
# 题目链接：

http://acm.timus.ru/problem.aspx?space=1&num=1523

# 题目大意：

给出 N 个数，求有多少个长度为 K 的严格递减子序列。

$1 \le N \le 20000$，$2 \le k \le 10$

# 题目分析：

首先为了方便处理，我们翻转一下数组，这样答案变成了求递增。

定义状态 $dp[i][j]$的含义为以 i 结尾的长度为 j 的递增子序列个数。

那么可以这样转移 :

$$dp[i][j] = \sum^{i - 1}_{k = 1} dp[k][j - 1] (\text {where $a_k < a_i$} )$$

这里我们依次计算长度$1 - k$的递增子序列，这样对长度为 j 的子序列就可以从左到右遍历数组，用树状数组统计长度为 j - 1 的子序列贡献。


# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 21234;
const int MOD = 1e9;

int data[MAX], tree[MAX], dp[MAX][20];

inline int low_bit(int x) {
    return x&(-x);
}

int n, k;

//给 x 这个位置增加 v
void add(int x, int v) {
    while (x <= n) {
        tree[x] = (tree[x] + v) % MOD;
        x += low_bit(x);
    }
}

//求 1 ～ x 的和
int query(int x) {
    int rst = 0;
    while (x) {
        rst = (rst + tree[x]) % MOD;
        x -= low_bit(x);
    }
    return rst;
}

int main() {
    scanf("%d %d", &n, &k);
    for (int i = n; i >= 1; i--) {
        scanf("%d", data + i);
        dp[i][1] = 1;
    }
    for (int i = 2; i <= k; i++) {
        memset(tree, 0, sizeof(tree));
        for (int j = 1; j <= n; j++) {
            dp[j][i] = query(data[j]);
            add(data[j], dp[j][i - 1]);
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) ans = (ans + dp[i][k]) % MOD;
    printf("%d\n", ans);
}
```
# 解题过程：

树状数组还是挺好用的，这种问题写线段树还是太麻烦了...