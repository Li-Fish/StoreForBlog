---
title: LightOJ1370 - Bi-shoe and Phi-shoe（欧拉函数+打表）
date: 2017-07-20 16:51:12
categories: [ACM, 数学, 数论]
tags:
---
# 题目链接：
https://vjudge.net/problem/LightOJ-1370

---------------------------
# 题目大意：
给出 $N$ 个数$a_1,a_2\dots a_n$，求对每一个 $a_i$ 找出最小的 $k_i$ 使得 $\phi(k_i) \ge a_i$，输出 $\sum_1^n k_i$ 。

------------------------------
# 解题过程：

因为是数论的题，显然题目是要用欧拉函数，于是特意去翻了一下紫书的欧拉函数。想假期在家里做的，然后咸鱼了，想在来学校补得，也算是目前数论里面唯一一个自己做出的题了...

-------------------------
# 题目分析：

首先打表求出欧拉函数的值，用紫书上类似素数筛的方式，可以$O(n\log \log n)$的时间内求出。

然后需要解决的问题是，假设对于 2 这个数，那么用欧拉函数值为 $2$ 的 $k$ 并不一定是最优的，可能有一个 $m < k$ 但是 $\phi(m) > \phi(k)$。
这里一个解决方式是，找出每一个$a_i$的$k_i$打一张表，从1开始枚举$k$，先枚举到的k一定比后枚举到的优。

----------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 1123456;

typedef long long LL;

int phi[MAX];
int cost[MAX];

void init() {
    //对欧拉函数打表
    phi[1] = 0;
    for (int i = 2; i <= MAX; i++) {
        if (phi[i]) continue;
        for (int j = i; j <= MAX; j+= i) {
            if (!phi[j]) phi[j] = j;
            phi[j] = phi[j] / i * (i-1);
        }
    }

    //枚举每一个的花费k
    for (int i = 1; i <= MAX; i++) {
        int x = phi[i];
        for (int j = x; j >= 1; j--) {
            //如果当前为空，那么他的花费为i
            //否则之前已经有更优的解，直接break
            if (cost[j]) break;
            cost[j] = i;
        }
    }
}

int main() {
//    freopen("out", "w", stdout);
    init();
    int T;
    scanf("%d", &T);
    for (int cases = 1; cases <= T; cases++) {
        int n;
        LL ans = 0;
        scanf("%d", &n);
        while (n--) {
            int t;
            scanf("%d", &t);
            ans += cost[t];
        }
        printf("Case %d: %lld Xukha\n", cases, ans);
    }

}
```