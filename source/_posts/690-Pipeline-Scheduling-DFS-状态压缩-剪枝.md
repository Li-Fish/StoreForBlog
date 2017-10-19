---
title: 690 - Pipeline Scheduling (DFS + 状态压缩 + 剪枝)
date: 2017-03-15 18:07:24
categories: [ACM, 搜索]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=631

# 题目大意
------------------
有 5 个不同的工作单元，10 个相同的程序，每个程序需要运行 n 个时间段，每个时间段需要一个工作单元工作。现在问至少需要多少时间，才可以执行完全部的程序。

# 解题过程
--------------
大体思路就是暴力模拟，显然可以状态压缩用位运算，然后就是剪枝的问题了。

最开始想的是最暴力的模拟，从移一位开始试，如果可以移动了就 break 掉，这样找的每一步都是最短的，但总体上不是最优的。

然后就去删掉了 break ，循环到移动 n 位。

最后还是超时，又加了如果当前长度加上后面的最少长度大于答案就剪掉，还是超时，不过这里到是用的 IDA* 换了直接 DFS 还是超时。

于是去找了博客……

# 题目分析
------------------
#### 以后做题的时候要考虑下，暴力 dfs 时候，每一个状态到下一个状态的路径个数，可不可以优化，感觉这题的思想类似 KMP 先预处理可以走的步长，避免以后很多次的判断。

+ 有两次剪枝
	+ 一是开始就初始化一下，仅有两个任务的情况每次可以移动的步长。这样比直接暴力移动省了不少时间。
	+ 二是如果当前还需要运行的程序个数乘最短步长后，如果大于当前的结果，那么剪掉。
+ 每次的状态用一个一位数组储存，每个工作单元用状态压缩一个整数表示。
+ 每次检测好移动用位运算。
# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 30;

char rawData[6][MAX];
int data[MAX], n, ans, jump[MAX], cnt;

bool judge(int A[], int k) {
    for (int i = 0; i < 5; i++) {
        if ((A[i]>>k)&data[i])
            return false;
    }
    return true;
}

void codingData(int n) {
    ans = n*10, cnt = 0;
    memset(data, 0, sizeof(data));
    for (int i = 0; i < 5; i++) {
        scanf("%s", rawData[i]);
        for (int j = 0; j < n; j++) {
            if (rawData[i][j] == 'X') {
                data[i] |= (1<<j);
            }
        }
    }
    for (int i = 0; i <= n; i++) {
        if (judge(data, i))
            jump[cnt++] = i;
    }
}

void attempt(int dep, int A[], int pos) {
    if (pos + (10 - dep) * jump[0] > ans)
        return;
    if (dep == 10) {
        ans = min(pos, ans);
        return;
    }
    for (int i = 0; i < cnt; i++) {
        if (judge(A, jump[i])) {
            int B[MAX] = {};
            for (int j = 0; j < 5; j++) {
                B[j] = (A[j]>>jump[i]) | data[j];
            }
            attempt(dep+1, B, pos + jump[i]);
        }
    }
}

int main() {
    while (~scanf("%d", &n) && n) {
        codingData(n);
        attempt(1, data, n);
        printf("%d\n", ans);
    }
}

```