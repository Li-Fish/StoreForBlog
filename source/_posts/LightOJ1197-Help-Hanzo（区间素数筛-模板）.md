---
title: LightOJ1197 - Help Hanzo（区间素数筛 + 模板）
date: 2017-07-21 18:01:29
tags:
categories: [ACM, 数学, 数论]
---
# 题目链接：
https://vjudge.net/problem/LightOJ-1197

---------------------------
# 题目大意：
给出$a,b$求$[a,b]$内素数个数，保证$b - a < 100000$，$1\le a\le b<2^{31}$。

----------------------------
# 解题过程：
这题有点可惜，没仔细想就去翻书了，挑战第二版 P121，当初这里看过了，以为只有一个埃氏筛法然后跳过了，有点可惜...

-----------------------------
# 题目分析：
首先对于任意的$b$，他的最小质因数一定不会大于$\sqrt {b}$，那么可以用$[2, \sqrt b]$的素数表，去筛掉$[a, b]$区间内的倍数，剩下的就是素数了。

然后用了两种差不多的实现。


这题要注意$1$不是素数！！！！！

----------------------------
# AC代码一：
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAX = 100000;
const int MAX_INTERVAL = 200000+100;

vector<int> prime;
bool not_prime[MAX];
bool interval[MAX_INTERVAL];

//先筛出小于等于sqrt(b)的素数
void init() {
    for (int i = 2; i < MAX; i++) {
        if (not_prime[i]) continue;
        prime.push_back(i);
        for (int j = i << 1; j < MAX; j += i)
            not_prime[j] = true;
    }
}

int solve(ll a, ll b) {
    memset(interval, 0, sizeof(interval));
    for (int i = 0; i < prime.size(); i++) {
        if ((ll)prime[i] * prime[i] > b) break;
        
        //计算起始的位置，区间内第一个prime[i]的倍数
        ll s = (a + prime[i] - 1) / prime[i];
        //如果算出的s是1的话，那么就是这个素数本身，这种情况不应该筛掉
        if (s < 2) s = 2;
        
        s *= prime[i];
        for (; s <= b; s += prime[i]) {
            interval[s-a] = true;
        }
    }
    
    //最后统计答案
    int ans = 0;
    for (int i = 0; i <= b - a; i++) {
        if (!interval[i]) ans++;
    }
    return ans;
}

int main() {
    int T;
    init();
    scanf("%d", &T);
    for (int Case = 1; Case <= T; Case++) {
        ll a, b;
        scanf("%lld %lld", &a, &b);
        int ans = solve(a, b);
        if (a == 1) ans--;
        printf("Case %d: %d\n", Case, ans);
    }
}
```

---------------------
# AC代码二：
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

const int MAX = 100000;
const int MAX_INTERVAL = 200000+100;

//这个是最终答案的区间
bool not_prime[MAX_INTERVAL];
//这个是小于sqrt(b)的区间
bool not_prime_small[MAX];

int solve(ll a, ll b) {
    memset(not_prime, 0, sizeof(not_prime));
    memset(not_prime_small, 0, sizeof(not_prime_small));

    //枚举所有小于等于sqrt(b)的数
    for (int i = 2; (ll) i * i < b; i++) {
        //如果是素数进行下一步
        if (!not_prime_small[i]) {
            //筛掉小区间内的倍数
            for (int j = 2 * i; (ll)j * j < b; j += i) {
                not_prime_small[j] = true;
            }
            //筛掉大区间内的倍数，找起始位置的方法与第一种一样
            for (ll j = max(2LL, (a + i - 1) / i) * i; j <= b; j += i) {
                not_prime[j - a] = true;
            }
        }
    }
    int ans = 0;
    for (int i = 0; i <= b - a; i++) {
        if (!not_prime[i]) ans++;
    }
    return ans;
}

int main() {
    int T;
    scanf("%d", &T);
    for (int Case = 1; Case <= T; Case++) {
        ll a, b;
        scanf("%lld %lld", &a, &b);
        int ans = solve(a, b);
        if (a == 1) ans--;
        printf("Case %d: %d\n", Case, ans);
    }
}
```