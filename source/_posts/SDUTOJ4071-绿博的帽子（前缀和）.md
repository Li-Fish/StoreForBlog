---
title: SDUTOJ4071 - 绿博的帽子（前缀和）
date: 2017-11-20 12:21:22
categories: [ACM, 命题]
tags:
---
# 题目链接：

http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/4071.html

# 题目大意：

<<<<<<< HEAD
给出 n 个数 $a_1, a_2, a_3 \dots a_n$，进行 q 次询问，每次输出 $\sum_{k = l}^r $$a_k$。
=======
给出 n 个数 $a_1, a_2, a_3 \dots a_n$，进行 q 次询问，每次输出 $\sum_{k = l}^r a_k$。
>>>>>>> ced470192faae13e215fb3b57e28efad2f01fa7f

$1 \le n, m, l, r \le 100000, l \le r \le n$



# 题目分析：

题意很简单，但是如果直接用 for 循环去求 l 到 r 的和的话会超时，可以计算一下，最多有 100000 次询问，每次最差计算 100000 次，这样总共时间复杂度是 $10^{10}$，计算机一秒大概只能计算$10^{9}$次，所以肯定会超时。

<<<<<<< HEAD
这时候我们定义$S_i = \sum_{k=1}^i ​$ $a_k​$，这样我们可以用一个 for 循环求出所有的 Si，这样对于每一个 l, r 的询问，我们用 $S_r​$ $- S_{l - 1}​$即是答案，这样总共的时间复杂度是 $10^5​$。
=======
这时候我们定义$S_i = \sum_{k=1}^i a_k$，这样我们可以用一个 for 循环求出所有的 Si，这样对于每一个 l, r 的询问，我们用 $S_r - S_{l - 1}$即是答案，这样总共的时间复杂度是 $10^5$。
>>>>>>> ced470192faae13e215fb3b57e28efad2f01fa7f




# AC代码：
```cpp
#include <stdio.h>

const int MAX = 112345;
int S[MAX];

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n;
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) {
            scanf("%d", S + i);
        }
        for (int i = 1; i <= n; i++) {
            S[i] += S[i - 1];
        }
        int q;
        scanf("%d", &q);
        while (q--) {
            int l, r;
            scanf("%d %d", &l, &r);
            printf("%d\n", S[r] - S[l - 1]);
        }
    }
} 
```