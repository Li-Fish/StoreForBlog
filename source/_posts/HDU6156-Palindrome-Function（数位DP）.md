---
title: HDU6156 - Palindrome Function（数位DP）
date: 2017-08-25 10:37:05
categories: [ACM, DP, 数位DP]
tags:
---
# 题目链接：

http://acm.hdu.edu.cn/showproblem.php?pid=6156



--------------------
# 题目大意：

设函数 f(n, k) 的取值为，若 n 在 k 进制下为回文数字，那么函数值为 k 否则 为 1 。

给出 a ，b ，L， R。

求 $\sum\_{i=a}^b$ $ \sum\_{j = L}^R$ $f(i, j)$ 。



-------------------
# 解题过程：

CCPC 网络赛的题，算是一个裸的板子题了，数位 DP 之前也算是做过，不过没做过这种类型的数位 DP ，然后弃疗了。过了好久把数位 DP 的专题刷掉之后才来补的。



--------------------
# 题目分析：

定义状态`dp[start][pos][flag][k]`

表示在 k 进制下以 start 位置开始的回文串，在 pos 位置下，回文串个数，flag 表示当前串是否为回文串。

对于前导零是忽略掉的，选取前导零就视为将 start 位置减一。对于回文串，前一半随意枚举就可以，后一半要进行判断。即对 `(start+1)/2 > pos `的情况进行判断，这时候要用一个数组记录之前枚举的数字。



----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long LL;

const int MAX = 50;

LL dp[MAX][MAX][2][MAX];
int num[MAX], tmp[MAX], k;

LL dfs(int cur, int start, bool flag, bool bound) {
    if (cur < 0) return flag;
    if (!bound && dp[cur][start][flag][k] != -1) return dp[cur][start][flag][k];
    int last = bound ? num[cur] : k - 1;
    LL ans = 0;
    for (int i = 0; i <= last; i++) {
        //判断是否为前导零
        bool st = (cur == start && i == 0);
        bool new_flag = flag;
        if (flag) {
            //如果当前是回文串的后半段的话，就判断下当前是否构成回文串
            if (!st && cur < (start + 1) / 2) new_flag = (tmp[start - cur] == i);
        }
        tmp[cur] = i;
        ans += dfs(cur - 1, st ? start - 1 : start, new_flag, bound && (i == last));
    }
    if (!bound) dp[cur][start][flag][k] = ans;
    return ans;
}

LL solve(int n) {
    if (n == 0) return 1;
    int len = 0;
    while (n) {
        num[len++] = n % k;
        n /= k;
    }
    return dfs(len - 1, len - 1, true, true);
}

int main() {
    int T;
    scanf("%d", &T);
    memset(dp, -1, sizeof(dp));
    for (int Case = 1; Case <= T; Case++) {
        int a, b, l, r;
        scanf("%d %d %d %d", &a, &b, &l, &r);
        LL ans = 0;
        //枚举进制
        for (int i = l; i <= r; i++) {
            k = i;
            LL t = (solve(b) - solve(a - 1));
            ans += (b - a + 1) + t * (k - 1);
        }
        printf("Case #%d: %lld\n", Case, ans);
    }
}
```