---
title: CodeForces748E - Santa Claus and Tangerines(递推|二分DP)
date: 2017-08-19 10:32:15
categories: [ACM, DP]
tags:
---
# 题目链接：

http://codeforces.com/problemset/problem/748/E



--------------------
# 题目大意：

给出 n 个数，可以将一个数 k 拆成 k / 2 和 (k + 1) / 2，把这 n 个数至少拆成 k 个数，使得这 k 个数中最小的尽量的大。



-------------------
# 解题过程：

感觉就是二分答案 + DP，用 DFS 写的DP，结果 T 了好久，还以为是卡常数，试了好久，最后去翻了博客，才发现写成递推就稳了....

而且看到了一个神奇的做法。



--------------------
# 题目分析：

第一种做法是，二分答案，然后DP去验证是否有解。

dp[i] 表示最多可以得到多少个 i，然后对于所有的 $i / 2 \ge mid$，让 dp[i] 累加到 dp[i/2], dp[(i+1)/2] 上。对于 $i/2 < mid$ 直接统计到结果上，如果最后结果大于等于 k，表示有解。



第二种做法，直接枚举答案，并在一次递推中维护。

dp[i] 数组含义和上面一样，枚举到一个数字的时候，先让答案加上 dp[i]，这时候应该把 i 的父亲减去，因为 i 的父亲已经分解成 i 了，不应该对答案有贡献，但是只需要减去 $2i$和$2i-1$，就可以了因为 $2i+1$会在枚举 i+1 的时候减去。

注意对1特判一下，1没有$2i-1$这个父亲。



----------------------
# AC代码：

##做法一

```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 10000000 + 10;

int data[MAX];
long long dp[MAX];

int n, m, mid, mmm;

int Scan() {
    int res = 0, ch, flag = 0;
    if ((ch = getchar()) == '-')
        flag = 1;
    else if (ch >= '0' && ch <= '9')
        res = ch - '0';
    while ((ch = getchar()) >= '0' && ch <= '9')
        res = res * 10 + ch - '0';
    return flag ? -res : res;
}

bool ok() {
    long long rst = 0;
    memset(dp, 0, sizeof(dp));
    for (int i = 0; i < n; i++) {
        dp[data[i]]++;
    }
    for (int i = mmm; i >= mid; i--) {
        if (i / 2 >= mid && dp[i] > 0) {
            dp[i / 2] += dp[i];
            dp[(i + 1) / 2] += dp[i];
        } else rst += dp[i];
    }

    return rst >= m;
}

int main() {
    n = Scan();
    m = Scan();
    int l = 1, r = 0;
    for (int i = 0; i < n; i++) {
        data[i] = Scan();
        r = max(r, data[i]);
    }
    int ans = r >= m ? 1 : -1;
    mmm = r;
    while (l <= r) {
        mid = (l + r) / 2;
        if (ok()) {
            ans = mid;
            l = mid + 1;
        } else r = mid - 1;
    }

    printf("%d\n", ans);
}
```
##做法二

```cpp
#include <bits/stdc++.h>

using namespace std;

const int maxn = 1000000;

int a[maxn + 1], n, m, mx;

long long dp[maxn * 10 + 1], cur;

int main() {
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%d", &a[i]), cur += a[i], dp[a[i]]++, mx = max(mx, a[i]);
    if (cur < m) return puts("-1"), 0;
    cur = 0;
    for (int i = mx; i; i--) {
        cur += dp[i];
        //减去父亲贡献
        if (i * 2 <= mx) cur -= dp[i << 1];
        if (i * 2 - 1 <= mx && i != 1) cur -= dp[i * 2 - 1];
        //找到的第一个符合条件的答案即是最大值
        if (cur >= m) return printf("%d\n", i), 0;
        //对当前数拆分
        dp[i >> 1] += dp[i];
        dp[i + 1 >> 1] += dp[i];
    }
}
```

