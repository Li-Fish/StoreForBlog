---
title: SDUTOJ - 2781 二分练习（二分）
date: 2017-04-17 16:21:20
categories: [ACM, 二分]
tags:
---
# 题目链接：
http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/2781.html

-------------------------------
# 题目大意：
 给你一个序列，然后给你m个元素，让你从序列中找出与每个元素最接近的数字输出来，如果有两个就输出两个。

--------------------------------
# 解题过程：
 刚开始是 WA 了好久，看了博客，听学长讲完才知道做法，这里当作一个二分的模板。

----------------------
# 题目分析：
+ 本题主要是找二分的下界和上界。
+ 假设要查找一个数 n，上界是大于等于 n 的数中最小的。下界是小于等于 n 的数中最大的。

------------------------------
# AC代码：
```cpp
#include<bits/stdc++.h>
using namespace std;

int data[10000000+10];

int find_left(int l, int r, int k) {
    int rst = 0;
    while (l <= r) {
        int m = (l+r) >> 1;
        if (data[m] <= k) {
            rst = m;
            l = m+1;
        } else {
            r = m-1;
        }
    }
    return rst;
}

int find_right(int l, int r, int k) {
    int rst = 0;
    while (l <= r) {
        int m = (l+r) >> 1;
        if (data[m] >= k) {
            rst = m;
            r = m-1;
        } else {
            l = m+1;
        }
    }
    return rst;
}

int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m)) {
        for (int i = 0; i < n; i++) {
            scanf("%d", data+i);
        }
        sort(data, data+n);
        while (m--) {
            int k;
            scanf("%d", &k);
            int left = find_left(0, n-1, k);
            int right = find_right(0, n-1, k);

            if (abs(k-data[left]) != abs(k-data[right])) {
                if (abs(k-data[left]) < abs(k-data[right]))
                    printf("%d\n", data[left]);
                else
                    printf("%d\n", data[right]);
            }
            else if (data[left] == data[right]) {
                printf("%d\n", data[left]);
            }
            else {
                printf("%d %d\n", data[left], data[right]);
            }
        }
        putchar('\n');
    }
}
```
