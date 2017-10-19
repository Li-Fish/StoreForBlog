---
title: UVa 11426 - GCD - Extreme (II)
date: 2017-07-25 08:13:04
categories: [ACM, 数学, 数论]
tags:
---
# 题目大意：

输入$N$对公式$ \sum ^{n-1} \_{i=1} \sum ^{n} \_{j=i+1} GCD(i, j) $求和。

-------------------
# 解题过程：

一开始就没思路，然后去翻博客，才发现欧拉函数还可以这么用！

-----------------------
# 题目分析：

首先对内外两层循环交换下：

$\sum^{N-1}\_{i=1} \sum^{N}\_{j=i+1} GCD(i, j) = \sum^{n}\_{j=2}\sum^{j-1}\_{i=1}GCD(i,j)$

然后我们把内层循环拆出来，其实就是$f(n) = \sum^{n-1}_{i=1}GCD(i, n)$这样问题就是如何对这个函数求和了。

这时候，如果对每个数都分别跑一个循环肯定是超时，那么我们枚举每个数的因素，并且用素数筛的思想。

首先，记$g(i, n)$ 为满足 $gcd(x, n) = i(i \le x < n)$的$x$个数，那么对于每个数$f(n) = \sum g(i, n) \cdot i$。如何对于$gcd(x, n) = i$两边同时除去$i$，$gcd(x/i, n/i) = 1$，即$g(i, n) = \phi(n/i)$。

--------------------------
# AC代码:
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAX = 4000001 + 10;

int phi[MAX];
ll ans[MAX];

//欧拉函数打表
void phi_table(int n) {
    memset(phi, 0, sizeof(phi));
    phi[1] = 1;
    for (int i = 2; i < n; i++) if (!phi[i])
            for (int j = i; j <= n; j += i) {
                if (!phi[j]) phi[j] = j;
                phi[j] = phi[j] / i * (i - 1);
            }
}

void init() {
    //枚举每个数的因数
    for (int i = 1; i < MAX; i++) {
        for (int j = i + i; j < MAX; j += i) {
            //这里求的是f(j)的其中一个因子
            ans[j] += (ll)i * phi[j / i];
        }
    }
    for (int i = 2; i < MAX; i++) {
        ans[i] += ans[i-1];
    }
}

int main() {
    phi_table(MAX);
    init();
    int n;
    while (cin >> n && n) {
        cout << ans[n] << endl;
    }
}
```