---
title: Gym 100917J Judgement(背包DP+bitset)
date: 2017-08-21 21:02:23
categories: [ACM, DP]
tags:
---
# 题目链接：

https://vjudge.net/problem/Gym-100917J



--------------------
# 题目大意：

给出两个长度为 n 序列 $ A\_ {i}, B\_ {i} $ 和 p , q。如果存在一个集合$c\_ 1,c\_ 2,c\_ 3 \dots c\_ k$，使得$(\sum A\_ {c\_ i} \ge p \wedge \sum B\_ {c\_ i} < q) \bigvee (\sum A\_ {c\_ i} < p \wedge \sum B\_ {c\_ i}  \ge q)$ 那么输出 NO，并用 01 输出集合中的元素，全集为1~n。



-------------------
# 解题过程：

比赛的时候想用贪心暴力水一发的，居然水道37组样例，结果还是不对，赛后补的，也算是学下 bitset 了。



--------------------
# 题目分析：

跑两次 DP，定义 dp[i] = j 的含义为，第一个序列的和为 i 时，第二个序列的最大值为 j。

我们只跑 i < p 的，如果存在 j >= q，那么就不符合上面的条件了。

然后对第一个序列第二个序列交换位置，再跑一次。

最后答案要输出路径，记录下最后从哪个状态转移而来的，并输出状态。



----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1123456;

int n;
int a[MAX], b[MAX];
bitset<120> fa[MAX];
int dp[MAX];

bool solve() {
    int p = a[0], q = b[0];
    memset(dp, -1, sizeof(dp));
    for (int i = 0; i < MAX; i++) fa[i].reset();
    dp[0] = 0;
    for (int i = 1; i <= n; i++) {
        for (int j = p; j >= 0; j--) {
            if (dp[j] < 0) continue;
            if (j + a[i] < p && dp[j + a[i]] < dp[j] + b[i]) {
                dp[j + a[i]] = dp[j] + b[i];
                fa[j + a[i]] = fa[j];
                fa[j + a[i]].set(i);
                if (dp[j + a[i]] >= q) {
                    puts("NO");
                    for (int k = 1; k <= n; k++) {
                        printf("%d", (int)fa[j+a[i]][k]);
                    }
                    puts("");
                    return true;
                }
            }
        }
    }
    for (int i = 0; i <= n; i++) swap(a[i], b[i]);
    return false;
}

int main() {
    scanf("%d", &n);
    for (int i = 0; i <= n; i++) scanf("%d", &a[i]);
    for (int i = 0; i <= n; i++) scanf("%d", &b[i]);
    if (!(solve() || solve())) puts("YES");
}
```