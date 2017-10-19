---
title: URAL1183 - Brackets Sequence（区间DP）
date: 2017-10-16 09:01:19
categories: [ACM, DP, 区间DP]
tags:
---
# 题目链接：

http://acm.timus.ru/problem.aspx?space=1&num=1183

# 题目大意：

[参考博客](http://blog.csdn.net/jiange_zh/article/details/49994207)

> 定义正规的括号序列如下: 
>
> 1. 空序列是一个正规的括号序列 
> 2. 如果S是一个正规的括号序列, 那么(S) 和[S] 也都是正规的括号序列。 
> 3. 如果A和B是正规的括号序列, 那么AB也是一个正规的括号序列。 
>
> 现给定一个括号序列A（只包含小括号和中括号，可能为空序列），求一个正规括号序列B，使得A包含于B，而且B的长度是满足条件的序列中最小的。

# 题目分析：

设 $dp[i][j]$ 为使得 [i, j] 这段区间括号匹配所需要的最小花费，那么根据题意，$dp[i][j]$可由两种方式转移而来：

+ 如果 i 与 j 可以匹配的话$dp[i][j] = dp[i + 1][j - 1]$
+ 不关 i 与 j 是否匹配 $dp[i][k] = dp[i][k] + dp[k + 1][j]$



最后递归的打印答案，转移的时候标记一下，当前是否分为两个子序列。


# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int INF = 0x3f3f3f3f;

char s[1123];

int dp[112][112];
int mark[112][112];

void print(int l, int r) {
    if (l > r) return;
    if (l == r) {
        if (s[l] == '(' || s[l] == ')') cout << "()";
        else cout << "[]";
        return;
    }
    if (mark[l][r] == -1) {
        if (s[l] == '(') {
            cout << "(";
            print(l + 1, r - 1);
            cout << ")";
        } else {
            cout << "[";
            print(l + 1, r - 1);
            cout << "]";
        }
    } else {
        print(l, mark[l][r]);
        print(mark[l][r] + 1, r);
    }
}

int main() {
    gets(s + 1);
    int n = strlen(s + 1);
    if (!n) {
        cout << endl;
        return 0;
    }
    memset(dp, 0, sizeof(dp));
    for (int i = 1; i <= n; i++) dp[i][i] = 1;
    for (int l = 2; l <= n; l++) {
        for (int i = 1; i <= n - l + 1; i++) {
            int j = i + l - 1;
            dp[i][j] = INF;
            if (s[i] == '(' && s[j] == ')' || s[i] == '[' && s[j] == ']') {
                dp[i][j] = min(dp[i][j], dp[i + 1][j - 1]);
            }
            mark[i][j] = -1;
            for (int k = i; k < j; k++) {
                if (dp[i][k] + dp[k + 1][j] < dp[i][j]) {
                    dp[i][j] = dp[i][k] + dp[k + 1][j];
                    mark[i][j] = k;
                }
            }
        }
    }
    print(1, n);
    cout << endl;
}
```
# 解题过程：

这题卡了好久ORZ，之前了解过一点区间DP，结果还是不会做。