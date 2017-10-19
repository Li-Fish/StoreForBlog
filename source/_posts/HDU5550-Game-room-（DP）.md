---
title: HDU5550 - Game room （DP）
date: 2017-05-21 14:42:00
categories: [ACM, DP]
tags:
---
# 题目链接：

http://acm.hdu.edu.cn/showproblem.php?pid=5550

-----------------------------

# 题目大意：

 有一栋楼，有N层，每一层都有ai个想要玩A游戏的，bi个想要玩B游戏的，但是每层只能修建一种游戏厅。每个人移动上下一层楼需要消耗一点体力。使得所有人玩的上游戏并且消耗的体力尽量的少，最少消耗的体力。

---------------------------------

# 解题过程：

 比赛的时候好不容易读懂了题意，发现并不会做，第一个想法是贪心的，从0层向下扫，累加想玩A游戏的人数和想玩B游戏的人数，对于每一层判断是想玩A的人多还是想玩B的人多，修建人多的那个。

 显然这个思路得不到最优解，如果每个人只能向下走不能向上走的话应该可行。

 然后这个题放置了好长时间，现在才去补，翻了两三个博客算是看懂了，感觉这种DP只能靠脑洞了，每一种都不一样。


参考博客：
https://ramay7.github.io/2016/11/04/HDU-5550-2015CCPC-K-dp/

http://blog.csdn.net/snowy_smile/article/details/49618219

---------------------------------


# 题目分析：
 上面两个博客都说的非常详细，我主要是参照是第一个博客，代码加了注释，这里不做过多论述了。


----------------------------------
# AC代码：

```cpp
#include <cstdio>
#include <algorithm>
using namespace std;
typedef long long ll;
const int MAX_N = 4010;
const ll INF = 0x3f3f3f3f3f3f3f3fll;

int T, n, cases = 0;
ll value[MAX_N][2], sum[MAX_N][2], pre[MAX_N][2], suf[MAX_N][2];
ll dp[MAX_N][2];

void init() {
    //初始化求出前缀和，pre[i]表示从第i层移到第0层所需要的代价，suf[i]代表从第i层移到第n+1层所需要的代价
    for (int i = 1; i <= n; i++) {
        sum[i][0] = sum[i-1][0] + value[i][0];
        sum[i][1] = sum[i-1][1] + value[i][1];
        pre[i][0] = pre[i-1][0] + value[i][0] * i;
        pre[i][1] = pre[i-1][1] + value[i][1] * i;
    }
    for (int i = n; i >= 1; i--) {
        suf[i][0] = suf[i+1][0] + value[i][0] * (n - i + 1);
        suf[i][1] = suf[i+1][1] + value[i][1] * (n - i + 1);
    }
}

ll down(int a, int b, int id) {
    //表示 [a, b] 这个区间里的人要达到b+1所需要的代价
    return suf[a][id] - suf[b+1][id] - (sum[b][id] - sum[a-1][id]) * (n-b);
}

ll up(int a, int b, int id) {
    //表示 [a,b] 这个区间里的人要到达a所需要的代价
    return pre[b][id] - pre[a][id] - (sum[b][id] - sum[a][id]) * a;
}

ll work(int a, int b, int id) {
    int mid = (a+b) >> 1;
    //因为dp[i][0]表示的状态是当前是0，i+1是1，如果当前i为n，那么后面就没有1的房间了
    if (b < n) return up(a, mid, id) + down(mid+1, b, id);
    else return up(a, b, id);
}

void solve() {
    init();
    for (int i = 1; i <= n; i++) {
        dp[i][0] = dp[i][1] = INF;
        //计算前i个全为1或全为0的情况
        if (i < n) dp[i][0] = down(1, i, 1);
        if (i < n) dp[i][1] = down(1, i, 0);
        for (int j = 1; j <= i-1; j++) {
            //枚举上一个选1或0的位置
            dp[i][0] = min(dp[i][0], dp[j][1] + work(j, i, 1));
            dp[i][1] = min(dp[i][1], dp[j][0] + work(j, i, 0));
        }
    }
    printf("Case #%d: %lld\n", ++cases, min(dp[n][0], dp[n][1]));
}

int main() {
    scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%lld%lld", &value[i][0], &value[i][1]);
        }
        solve();
    }
}
```