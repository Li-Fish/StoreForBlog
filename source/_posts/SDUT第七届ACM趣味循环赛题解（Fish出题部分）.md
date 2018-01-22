---
title: SDUT第七届ACM趣味循环赛题解（Fish出题部分）
date: 2017-12-27 15:42:07
categories:
tags:
---
# yxc 的日常

非常简单的一道题。输入 N 个数，求和判断是否大于等于 25 即可。由于题意，数据的和保证不会超过 24，所以对每一组数据直接输出 NO 也可以 AC。



## 代码

```cpp
#include <stdio.h>
int main()
{
    int i,n,a,r=0;
    while(scanf("%d",&n)!=EOF)
    {
        r=0;
        for(i=0;i<n;i++)
        {
            scanf("%d",&a);
            r+=a;
        }
        if(r>=25)printf("YES\n");
        else printf("NO\n");
    }
    return 0;
}
```



# 金泽的地图

这个题数据量非常大，有两种操作，一种 L 到 R 区间每个位置加 V，一种是询问 L 到 R 的和，操作次数不会超过 $10^5$，保证全部增加完再询问。

$1 \le L \le R \le 10^5$，对于每个增加或询问操作直接 for 循环去跑肯定会 TLE，由之前[绿博的帽子](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/4071.html)那个题可以知道对于询问我们可以用前缀和去预处理一下，然后每次询问都直接计算出来。

但是对于增加操作，我们也需要进行优化，我们这样处理：

首先定义一个初始值为 0 的数组 data

对于每一个 L 到 R 区间增加 V 的操作，让 `data[L] += v`，`data[R + 1] -= v`。

然后定义函数 $F(i) = \sum_1^idata[i]$，可以证明$F[i]$的值就是第 i 个位置进行所有增加的操作后最终的数值。



证明如下：

如果一次操作是 [L, R] 区间每个位置增加 V，那么我们实际进行的是 `data[L] += v`，`data[R + 1] -= v`。

对于位置 i，如果一次操作的区间 $L \le R < i$ 的话，即修改的区间的右端点小于 i 位置没有交集，那么 $\sum_1^i data[i]$ 中肯定同时包含了 data[L] 和 data[R + 1] 这两个值，然后对于这一次操作对$F(i)$的贡献为 $-v + v = 0$，是对$F(i)$的值无影响的。

如果一次操作的区间 $i < L \le R$ 的话，$\sum_1^idata[i]$，肯定不包含 data[L] 和data[R + 1] 这两个值，所以这次修改操作对$F(i)$的值无影响。

最后对于 $L \le i \le R$ 的区间，即 i 含于区间内，$\sum_1^i$ 肯定包含了 data[L] 但不包含 data[R + 1]，假设这次是增加 V，那么我们让`data[L] += v`，`data[R + 1] -= v`就让$F(i)$的值增加了 V，这正符合这个函数的定义。



最后我们对于每一个 L 到 R 区间增加 V 的操作，让 `data[L] += v`，`data[R + 1] -= v`。

定义 $F(i) = \sum_1^idata[i]$，$G(i) = \sum_1^iF(i)$

最后对于每次询问 L 到 R 的和，$G(R) - G(L - 1)$ 就是答案。



## 代码

```cpp
#include<iostream>
#include<cstring>

#define LL long long
using namespace std;
int n, m, i, r, l, x;
LL s[200000];

int main() {
    while (~scanf("%d%d", &n, &m)) {
        memset(s, 0, sizeof(s));
        for (i = 1; i <= n; i++) {
            scanf("%d%d%d", &l, &r, &x);
            s[l] += x;
            s[r + 1] += (-x);
        }
        for (i = 1; i <= 100000; i++) s[i] += s[i - 1];
        for (i = 1; i <= 100000; i++) s[i] += s[i - 1];
        for (i = 1; i <= m; i++) {
            scanf("%d%d", &l, &r);
            printf("%lld\n", s[r] - s[l - 1]);
        }
    }
    return 0;
}
```



# lxw AK

这题没什么难点，为了让大家了解一下 ACM-ICPC 比赛的罚时机制，按题意模拟即可。

关键是理解题意，AC 一道题后不增加罚时，一道题没有 AC 不计算罚时等。

## AC代码：

```cpp
#include <stdio.h>
#include <string.h>

#define MAX 100

int AC[MAX];
int num[MAX];

char pid, rst[20];
int t;

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, m;
        memset(AC, 0, sizeof(AC));
        memset(num, 0, sizeof(num));
        scanf("%d %d", &n, &m);
        int sum = 0;
        for (int i = 1; i <= m; i++) {
            scanf(" %c %s %d", &pid, rst, &t);
            pid -= 'A';
            if (!AC[pid]) {
                if (rst[0] == 'A') {
                    sum += t + num[pid] * 20;
                    AC[pid] = 1;
                } else {
                    num[pid]++;
                }
            }
        }
        printf("%d\n", sum);
    }
}
```



# 怪盗的谜题

这个题主要考察下将数字转化为字符串，并且判断字符串的回文。最后答案要预处理一下前缀和。



```cpp
#include <stdio.h>

#define MAX 100000 + 100

int data[MAX];
int num[100];

int get(int x) {
    int top = 0;
    while (x) {
        num[top++] = x % 10;
        x /= 10;
    }
    int l = 0, r = top - 1;
    int flag = 1;
    while (l <= r) {
        if (num[l] != num[r]) flag = 0;
        l++, r--;
    }
    int rst1 = 0, rst2 = 1;
    for (int i = 0; i < top; i++) {
        rst1 += num[i];
        rst2 *= num[i];
    }
    return flag ? rst2 : rst1;
}

int main() {
    for (int i = 1; i < MAX; i++) {
        data[i] += data[i - 1] + get(i);
    }
    int QAQ;
    scanf("%d", &QAQ);
    while (QAQ--) {
        int l, r;
        scanf("%d %d", &l, &r);
        printf("%d\n", data[r] - data[l - 1]);
    }
}
```

