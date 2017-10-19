---
title: CodeChef MOU2H - Mountain Holidays 2 （DP）
date: 2017-06-25 11:09:59
categories: [ACM, DP]
tags:
---
# 题目链接：
https://www.codechef.com/problems/MOU2H

-------------------------
# 题目大意：
理解题意后就是求一个序列中有多少个不同的子序列。

---------------------------
# 解题过程：

刚开始看错了题意，样例过不去，后来去翻了博客，才看懂题意，看懂题意后就好做了，就是一个简单的动态规划。

---------------------
# 题目分析：
因为要求不同子序列的个数。

定义状态$dp[i]$为前[i]个数中，不同子序列的个数。那么对于dp[i]可以由已下方式转移而来，记$pre[A[i]]$为A$[i]$这个数字上次出现的下标，如果未出现为$-1$。

定义$dp[0]$为$1$代表一个空串。

那么dp[i]可由以下状态转移而来：

$$
dp[i] = 	
\begin{cases}
dp[i-1]\times 2 , &pre[A[i]] = -1 \\
dp[i-1] \times 2 - dp[pre[A[i]]-1], &pre[A[i]] \neq -1
\end{cases}
$$


如果前i-1个数的不同子串个数为N，那么加上第i个数之后，对于前i个不同的子串加上第i个数后都构成了一个新的串，那么对于前i个数的不同子串为，前i-1的不同子串个数+新构成的子串个数。

不过如果第i个数曾经出现过的话，需要去重处理，如果3这个数字，在$5$和$9$这个位置都出现过的话，那么前$4$个数的子串后面加上第$5$个数和加上第$9$个数，构成的子串相同，这里需要减去最近出现的前一个位置的不同子串个数。
这里需要仔细理解下，$dp[i]$代表的是前$i$个元素构成的不同子串的个数，不是以元素$i$结尾的最大子串个数。




这里介绍一个骚操作，数组的下标可以为负数，对于下面代码。
```cpp
int a[3] = {1, 2, 3};
int *p = a+1;
cout << p[-1] << endl;
```
输出的结果为`1`

------------------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1123456;
const int INF = MAX<<2;
const int MOD = 1000000009;

int H[MAX];
int reserve[MAX*10], *pre = reserve+INF;
int dp[MAX];

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n;
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%d", H+i);
        }
        for (int i = 1; i < n; i++) {
            H[i] = H[i+1] - H[i];
            pre[H[i]] = -1;
        }
        dp[0] = 1;
        for (int i = 1; i < n; i++) {
            dp[i] = (dp[i-1]<<1)%MOD;
            if (pre[H[i]] != -1) {
                dp[i] = (dp[i] - dp[pre[H[i]]-1] + MOD)%MOD;
            }
            pre[H[i]] = i;
        }
        printf("%d\n", (dp[n-1]-1+MOD)%MOD);
    }
}


```