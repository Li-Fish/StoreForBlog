---
title: 12558 - Egyptian Fractions (HARD version) （IDA* + 剪枝）
date: 2017-03-16 21:55:23
categories: [ACM, 搜索]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4003
# 题目大意
------------------
埃及分数问题，给定一个分数，用几个不相同的分数表示，有多个表示的话，用的分数越少越好，还是多解的话，最小的分数越大越好，然后第二小的分数越大越好……一直到最大的分数越大越好。
# 解题过程
--------------
见紫书分析
# 题目分析
------------------
见紫书分析
代码已加入注释
# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

#define LL long long

LL ans[11234567], tans[11234567];

//book用来标记不可用作分母的数字，题目说小于1000，下面代码用到book的地方都是判断分母是否可用
bool book[11234];

//得到一个结果后，如果比ans更优，那么更新
bool better(int d) {
    for (int i = d; i >= 0; i--) {
        if (ans[i] != tans[i])
            return ans[i] == -1 || tans[i] < ans[i];
    }
}

//获得下一个比 a/b 小的 1/c
LL get_next(LL a, LL b) {
    for (LL i = 1; ; i++) {
        if (i <= 10000 && book[i])
            continue;
        if (b <= a * i)
            return i;
    }
}

bool dfs(int max_deep, LL from, LL aa, LL bb, int deep) {
    //如果到达最大深度，那么需要判断下，然后return
    if (deep == max_deep) {
        //判断最后的状态是否为 1/c 的形式，如果不算，那么当前状态不可表示原分数
        if (bb % aa)
            return false;
        if (bb <= 10000 && book[bb/aa])
            return false;
        tans[deep] = bb/aa;
        //如果当前解更优，那么更新ans
        if (better(deep))
            memcpy(ans, tans, sizeof(LL) * (deep+1));
        return true;
    }

    bool ok = false;

    //得到的 1/c 中 c 至少比上一次的分母大才可以 from 即是上一次的分母加一
    from = max(from, get_next(aa, bb));
    for (int i = from; ; i++) {
        if (i <= 10000 && book[i])
            continue;

        //如果剩下的递归次数，都是用 1/i 还小于 aa/bb 的话，那么当前 i 和比 i 大的数字不能用来做分母
        if (bb * (max_deep+1-deep) <= i * aa)
            break;
        tans[deep] = i;

        //计算 aa/bb - 1/i，gcd 用来约分
        LL b2 = bb*i;
        LL a2 = aa*i - bb;
        LL g = __gcd(a2, b2);
        if (dfs(max_deep, i+1, a2/g, b2/g, deep+1))
            ok = true;
    }
    return ok;
}

int main() {
    int n;
    scanf("%d", &n);
    for (int cases = 1; cases <= n; cases++) {
        memset(book, 0, sizeof(book));
        LL a, b, k;
        scanf("%lld %lld %lld", &a, &b, &k);
        for (int i = 0; i < k; i++) {
            LL t;
            scanf("%d", &t);
            book[t] = true;
        }

        for (int i = 0; ; i++) {
            memset(ans, -1, sizeof(ans));
            if (dfs(i,get_next(a, b), a, b, 0))
                break;
        }

        printf("Case %d: %lld/%lld=", cases, a, b);
        for (int i = 0; ans[i] != -1; i++) {
            if (i)
                putchar('+');
            printf("1/%lld", ans[i]);
        }
        putchar('\n');
    }
}

```