---
title: HDU5527 - Too Rich（贪心）
date: 2017-06-25 18:56:29
categories: [ACM, 贪心]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=5527

--------------------
# 题目大意：
现在你有$P$元钱，有$10$种不同面值的硬币，每种硬币有一定的数量，求用尽量多的硬币凑出P元钱，有可能凑不出。

-------------------------
# 解题过程：
当初比赛没看这道题，最近才补，看起来挺简单的，实际知道思路上也不难……

----------------------------
# 题目分析：

要用尽量多的硬币凑$P$元钱，假设我现在硬币总共的面值和为$M$，那么换个思路，我现在要从总共的硬币中拿走尽量少的硬币，使剩下的硬币为P元。

那么问题转化成了，用尽量少的硬币去凑出$M-P$元，这个题目就有点熟悉了，不过这里有两个特殊的地方，一个是每种硬币都有数量限制，一个是面值大的硬币并不是面值小的硬币的整数倍。

假设没有数量限制，那么就是简单的贪心，尽量用面值大的硬币去凑出$M-P$元就好了。但是这个题用这个思路的话，会有这种情况。

假设现在有$20,20,20,50 $这四个硬币，要去凑$60$元，如果以之前的思路贪心的话，那么是无解的，但是这里可以不用$50$元这个硬币，而用$3$个$20$元的去凑出$60$元。

那么对于这种情况，就要尝试一下，对于每种硬币，去掉一个这种硬币后的解。


------------------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long LL;

const LL INF = 0x3f3f3f3f3f3f3f3f;

LL coin[] = {1, 5, 10, 20, 50, 100, 200, 500, 1000, 2000};
LL num[10], c[10];

LL ans = INF;

void dfs(int i, LL sum, LL t) {
    //sum == 0 表示找到了一个解
    if (sum == 0) {
        ans = min(ans, t);
        return;
    }
    //sum == -1 时需要返回，已经没有面值更小的硬币了
    if (i < 0)
        return;

    //计算出来当前硬币最多取多少
    c[i] = min(num[i], sum/coin[i]);
    dfs(i-1, sum - coin[i] * c[i], t + c[i]);

    //尝试少拿一个硬币
    if (c[i] > 0) {
        c[i]--;
        dfs(i-1, sum - coin[i] * c[i], t + c[i]);
    }
}

int main() {
    LL n;
    scanf("%lld", &n);
    while (n--) {
        LL p;
        LL total, sum;
        total = sum = 0;
        scanf("%lld", &p);
        for (int i = 0; i < 10; i++) {
            scanf("%lld", num + i);
            //sum计算所有硬币总共的价值
            sum += coin[i]*num[i];
            //total计算总共的硬币数
            total += num[i];
        }

        //计算出需要拿走多少钱
        sum -= p;

        if (sum < 0) {
            printf("-1\n");
            continue;
        }

        ans = INF;
        dfs(9, sum, 0);

        //如果ans == INF 表示一个解都没有
        if (ans == INF)
            printf("-1\n");
        else
            printf("%lld\n", total - ans);
    }
}
```