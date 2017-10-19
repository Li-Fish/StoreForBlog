---
title: LightOJ1214 - Large Division（高精度取模 + 模板）
date: 2017-07-21 17:46:16
categories: [ACM, 数学]
tags:
---
# 题目链接：
https://vjudge.net/problem/LightOJ-1214

------------------
# 题目大意：
两个数$-10^{100}<a<100^{100}$, $b$ 为 32bit范围内(其实并不是)，需要用64位整型才可以。问$a$能否被$b$整除。

--------------------------
# 解题过程：
先用 Java 的大数类水过了，然后感觉应该用到数论的知识，想起来之前好像也有一道高精度取模的题，当初用 Python 水过去了，现在认真的学一下高精度取模。

---------------------------
# 题目分析：
显然这题正负号和是否整除无关，先忽略掉。

对于每一个整数，可以分解为如下的形式（以$1234$举例）：
$12345 = ((1\times 10 + 2)\times 10+3)\times10 + 4$
然后这里可以顺带取模（以模123为例）：
$12345\mod 123 = (((1\times 10 \mod 123 + 2)\times 10 \mod 123+3)\times10 \mod 123+ 4)\mod123$

------------------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

char str[1123];
ll mod;

int main() {
    int T;
    scanf("%d", &T);
    for (int Case = 1; Case <= T; Case++) {
        scanf("%s %lld", str, &mod);
        int len = strlen(str);
        ll rst = 0;
        //忽略掉负号
        int i = str[0] == '-' ? 1 : 0;
        for (;i < len; i++) {
            rst = ((rst * 10) % mod + (str[i] - '0')) % mod;
        }
        //取模为0表示可以被整除
        if (rst == 0) {
            printf("Case %d: divisible\n", Case);
        }
        else {
            printf("Case %d: not divisible\n", Case);
        }
    }
}
```