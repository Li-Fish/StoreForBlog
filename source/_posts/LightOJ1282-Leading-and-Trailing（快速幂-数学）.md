---
title: LightOJ1282 - Leading and Trailing（快速幂+数学）
date: 2017-07-20 17:22:11
categories: [ACM, 数学]
tags:
---
# 题目链接：
https://vjudge.net/problem/LightOJ-1282

----------------------
# 题目大意：
求$n^k$的前3位数和后三位数。$2\le a<2^{31} , 1\le k \le 10^7$。

--------------------------
# 解题过程：

只让求后三位的话我到是会，用快速幂就好了，但是求前三位感一脸懵逼。于是去翻了博客，发现居然还有这种操作！

--------------------------------
# 题目分析：
后三位直接用快速幂取膜就好了，这里说一下前三位。

这里先假设：$n^k = a.bc...\times 10^m$，即用科学计数法表示，因为只要前三位，那么接下来就忽略掉后面的位。

对于上式两边同时取$\lg$：$k\lg n = \lg a.bc + m$
这里$m$一定为一个整数，$a.bc$在科学计数法中小于10，那么$\lg a.bc$一定为一个小于$0$的小数。

那么$\lg a.bc$ 为 $k\lg n$ 的小数部分，$m$为$k\lg n$的整数部分。

然后$abc = 10^{\lg a.bc} \times 100$，即为所求的前三位数。

------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int mod = 1000;

//快速幂+取模
int pow_mod(int n, int m) {
    int ans = 1, x = n;
    while (m > 0) {
        if (m&1) {
            ans = (ll)ans * x % mod;
        }
        x = (ll) x * x % mod;
        m >>= 1;
    }
    return ans;
}

int main() {
    int T, n, m;
    cin >> T;
    for (int Case = 1; Case <= T; Case++) {
        cin >> n >> m;

        //计算前三位数
        double t = log10(n) * m;
        t -= floor(t);
        int ans1 = pow(10, t)*100;

        int ans2 = pow_mod(n, m);
        printf("Case %d: %d %03d\n", Case, ans1, ans2);
    }
}
```