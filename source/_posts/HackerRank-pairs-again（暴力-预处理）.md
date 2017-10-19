---
title: HackerRank - pairs-again（暴力+预处理）
date: 2017-06-26 11:13:15
categories: [ACM, 数学]
tags:
---
# 题目链接：
https://www.hackerrank.com/contests/w26/challenges/pairs-again

------------------
# 题目大意：
给定一个数n，问有多少对$a,b$满足$xa+by=n$至少有一个解，$a<b$并且$0 < b, 0 < y$，且$x,y$是整数。

-----------------------------
# 解题过程：
比赛时候的题，当初那场比赛运气还不错，这道题看到一堆人WA了60多发，于是没敢去做，后来去补了，也算是学下如何预处理约数。

--------------------------------
# 题目分析：

这题的时限很奇怪，都能跑到59秒。

这里要预处理出来前n个数的所有约数。然后枚举$a$和$x$，$n-xa$为$bx$，把$bx$的约数当做$b$，记录下$bx$大于$a$的约数，不过对于$ax+by = cx + by = n$的情况要去一下重。

预处理约数可以类比下欧拉筛法的思想。

----------------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAX = 312345;

vector<int> divisor[MAX];
int flag[MAX];

int main() {
    int n;
    cin >> n;
    //预处理前n个数的约数，类比欧拉筛法
    for (int i = 2; i < n; i++) {
        for (int j = i; j < n; j += i) {
            divisor[j].push_back(i);
        }
    }
    int ans = 0;
    memset(flag, 0, sizeof(flag));
    for (int a = 1; a < n; a++) {
        for (int xa = a; xa < n; xa += a) {
            int yb = n - xa;
            for (int i = 0; i < divisor[yb].size(); i++) {
                int t = divisor[yb][i];
                //让每个t对于每个a只用一次，并且保证t > a
                if (flag[t] == a || t <= a) continue;
                flag[t] = a;
                ans++;
            }
        }
    }
    cout << ans;
}
```