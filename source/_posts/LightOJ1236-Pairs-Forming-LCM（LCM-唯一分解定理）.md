---
title: LightOJ1236 - Pairs Forming LCM（LCM+唯一分解定理）
date: 2017-07-21 17:29:52
categories: [ACM, 数学, 数论]
tags:
---
# 题目链接：
https://vjudge.net/problem/LightOJ-1236

----------------------------
# 题目大意：
给定一个数$n$，求满足$i \le j < n \wedge lcm(i, j) = n$的$(i, j)$对总共有多少个。

------------------------------
# 解题过程：

想了一会...不会，看的博客，就当是个结论好了。

-------------------------
# 题目分析：
对于每一对$(i, j)$，可由唯一分解定理写成如下形式：
$n = p_1^{e_1} \cdot p_2^{e_2} \dots p_k^{e_k}$
$i = p_1^{a_1} \cdot p_2^{a_2} \dots p_k^{a_k}$
$j = p_1^{b_1} \cdot p_2^{b_2} \dots p_k^{b_k}$

要使得$lcm(i, j) = n$的充要条件是满足$max(a_i, b_i) = c_i$。
那么问题就转化成了找满足上述条件的$(a_i,b_i)$对数，即是 $\small{2} c_i + 1$。

再根据分步乘法算出的即是答案。不过这样对于除了$i = j$的时候，其他的都分别计算了$(i, j), (j, i)$，这里最后答案要除以二并加一。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 11234567;

typedef long long ll;

vector<int> prime;
bool not_prime[MAX];

void get_prime() {
    for (int i = 2; i < MAX; i++) {
        if (not_prime[i]) continue;
        prime.push_back(i);
        for (int j = i << 1; j < MAX; j += i) {
            not_prime[j] = true;
        }
    }
}

int main() {
    get_prime();
    int T;
    scanf("%d", &T);
    for (int Case = 1; Case <= T; Case++) {
        ll n;
        ll ans = 1;
        scanf("%lld", &n);
        //进行质因子分解，并计算，但是这里素数表只到sqrt(n)
        for (int i = 0; i < prime.size(); i++) {
            if (prime[i] > n) break;
            if (n % prime[i] != 0) continue;
            int cnt = 0;
            while (n % prime[i] == 0) {
                n /= prime[i];
                cnt++;
            }
            ans *= (2 * cnt + 1);
        }
        //如果n不为1，说明还剩下一个大于sqrt(n)的质因子，要当前的结果乘三
        if (n > 1) ans *= 3;
        ans = ans / 2 + 1;
        printf("Case %d: %lld\n", Case, ans);
    }
}
```