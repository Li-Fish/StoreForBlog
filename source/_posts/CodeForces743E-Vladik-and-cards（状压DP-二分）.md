---
title: CodeForces743E - Vladik and cards（状压DP + 二分）
date: 2017-08-18 10:56:01
categories: [ACM, DP, 状压DP]
tags:
---
# 题目链接：

http://codeforces.com/problemset/problem/743/E



--------------------
# 题目大意：

给出一个只含数字 1~8 序列，找出一个符合下面条件的最长子序列。

1. 选出的子序列，每种数字出现的次数差不能超过，没选视为出现 0 次。
2. 选出的子序列中，相同数字出现的位置要连续。


-------------------
# 解题过程：

补的好久之前的题，一开始题意都没看懂，然后去搜了下题意，顺便看到了二分+状压dp的标签，不过还是不会做。刚开始做的时候漏掉了第二个条件....

最后翻到了两个博客：

+ http://blog.csdn.net/mengxiang000000/article/details/53695321
+ http://www.cnblogs.com/Saurus/p/6183721.html


--------------------
# 题目分析：

首先因为每种序列的差最大不会超过 1，那么我们可以二分答案，枚举较长的连续数字长度 L 。

然后用 DP 判断一个解是否可行，并且算出来最优的答案。

假设我们现在限制最长的长度为 L，那么对于每种数字有 L 和 L-1 两种长度可选，显然选长度为 L 是比 L-1 更优的，现在我们要知道最多能选多少个长度为 L 的。

首先用一个集合记录那些数已经选完，那些还没选，由于只有 8 个数字，所以可以用状态压缩。

设二维的 DP 数组 $dp[i][j]$ 的含义是，只考虑序列里前 i 个数字，选取的状态为 j 的时，最多能选几个长度为 L 的连续数字。

这时候有状态转移方程：

$$dp[next(i, L-1)][s \cup k] = max(itself, dp[i][s]) $$

$$ dp[next(i, L)][s \cup k] = max(itself, dp[i][s])$$

$next(i, L)$ 表示从当前位置开始往后第 L 个数字 k 的位置，这个可以通过预处理获得。

这样就得出了当最长的连续数字长度为 L 时可以选多少个长度为 L 的连续数字。

设上面结果为 sum，那么 $sum \cdot L + (8 - sum) \cdot (L - 1)$ 就是原问题的答案。

不过二分枚举 L 的时候 L 最小为 2，因为我们默认长度为 L-1 的连续数字是合法的。

我们特殊处理一下 L = 1 的情况，判断下有多少个数字至少出现了一次即可。



----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int MAX = 1123;
const int INF = 0x3f3f3f3f;

vector<int> pos[10];

int n, ans;
int a[MAX], dp[MAX][1 << 8], cur[10];

bool ok(int L) {
    memset(cur, 0, sizeof(cur));
    memset(dp, -1, sizeof(dp));
    //初始合法的状态只有在字符串开始并且一个没放的情况
    dp[1][0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < (1 << 8); j++) {
            if (dp[i][j] < 0) continue;
            for (int k = 1; k <= 8; k++) {
                //如果当前 dp 数组的值为负数，说明当前状态是不可到达的，不进行往后转移
                if (j & (1 << (k - 1))) continue;
                if (pos[k].size() - cur[k] < L - 1) continue;
                dp[pos[k][cur[k] + L - 2]][j | (1 << (k - 1))] = max(dp[pos[k][cur[k] + L - 2]][j | (1 << (k - 1))],
                                                                     dp[i][j]);
                if (pos[k].size() - cur[k] < L) continue;
                dp[pos[k][cur[k] + L - 1]][j | (1 << (k - 1))] = max(dp[pos[k][cur[k] + L - 1]][j | (1 << (k - 1))],
                                                                     dp[i][j] + 1);
            }
        }
        //cur记录每个数字出现了多少次
        cur[a[i]]++;
    }
    int sum = -INF;
    for (int i = 1; i <= n; i++) sum = max(sum, dp[i][(1 << 8) - 1]);
    if (sum <= 0) return 0;
    ans = max(ans, sum * L + (8 - sum) * (L - 1));
    return 1;
}

int main() {
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    //记录每个出现的数字对应的下标，转移可以通过 vector 下标直接访问某个数字第 i 次出现的位置
    for (int i = 1; i <= n; i++) pos[a[i]].push_back(i);

    //二分枚举 L 且 L 至少为 2
    int l = 2, r = n;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (ok(mid)) l = mid + 1;
        else r = mid - 1;
    }

    //当答案为 0 的时候，考虑 L = 1 的情况
    if (ans == 0) {
        for (int i = 1; i <= 8; i++) {
            if (pos[i].size() > 0) ans++;
        }
    }
    cout << ans;
}
```