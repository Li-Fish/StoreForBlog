---
title: HDU6070 - Dirt Ratio（线段树+二分答案）
date: 2017-08-04 09:53:27
categories: [ACM, 二分]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=6070

--------------------
# 题目大意：
给出一个序列A，对于序列里面的每一个数字范围为1～n，要求找出一段连续子序列，Y为区间长度，X为区间不同数字的个数，使得X/Y尽量的小。

-------------------
# 解题过程：
多校赛的第四题，也算是A题比较快的一场，最后卡在了这个题上，感觉可以做的，但是不会。只想到了线段树，完全没往二分上面想，听说有$O(N^2)$卡过去的。第一次遇到这样固定一个区间边界的线段树题。

--------------------
# 题目分析：
首先X是肯定小于等于Y，那么答案的范围一定为[0~1]，这个题是特判，精度误差在$10^{-4}$就可以，这样二分答案只需要20次左右。

对于二分答案就是检查当前线段树上是否有一个区间，使得
$$ \frac {size(l, r)} {r-l+1}  \leq mid$$
$$ size(l, r) + l \times mid \leq mid \times (r + 1)$$

上述的size(l, r)为[l, r]内不同数字的个数

这么我们在线段树上维护左边的值，每次在线段树上查询是否有一段区间满足上述公式。

对于线段树上的每一个节点，假设维护的区间为$(a, b)$，那么维护的信息为$min(size(i, r) + l \times mid), a \ge i \le b$。

由于我们相当于固定住了r这个边界，对r从1～n开始枚举，对于每次对线段树新插入一个数x，假设x上一次出现的位置为p，当前插入的位置为r，那么只对 $(p+1,r]$这段区间内的size值有影响，需要加一。

另外在查询的时候，要保证查询的区间的右边界都要小于当前枚举到的r。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

#define root 1, 1, n
#define lson o<<1
#define rson o<<1|1
#define MID int m = (l+r)/2

using namespace std;

const int MAX = 60000 + 10;

struct Node {
    int lazy;
    double v;
} tree[MAX << 2];

int data[MAX], Case, n, last[MAX];
double M;

inline void tag(int o, int p) { tree[o].lazy += p, tree[o].v += p; }

inline void push_down(int o) {
    if (tree[o].lazy) { tag(lson, tree[o].lazy), tag(rson, tree[o].lazy), tree[o].lazy = 0; }
}

void merge(int o) {
    //更新根节点的最小值
    tree[o].v = min(tree[lson].v, tree[rson].v);
}

void build(int o, int l, int r) {
    //初始化假设区间内不同的数字为0
    tree[o].v = M * l;
    tree[o].lazy = 0;
    if (l == r) return;
    MID;
    build(lson, l, m);
    build(rson, m + 1, r);
}

void updata(int o, int l, int r, int ul, int ur) {
    if (ul > r || l > ur) return;
    if (ul <= l && r <= ur) {
        //区间内不同数字的数量加一
        tag(o, 1); return;
    }
    push_down(o);
    MID;
    updata(lson, l, m, ul, ur);
    updata(rson, m + 1, r, ul, ur);
    merge(o);
}

double query(int o, int l, int r, int pos) {
    //进行剪枝
    if (l > pos) return 1e9;
    //必须当前的右边界小于pos才能返回答案
    if (r <= pos) {
        return tree[o].v;
    }
    push_down(o);
    MID;
    return min(query(lson, l, m, pos), query(rson, m + 1, r, pos));
}

int main() {
    scanf("%d", &Case);
    while (Case--) {
        scanf("%d", &n);
        for (int i = 1; i <= n; i++) scanf("%d", data + i);
        double L = 0, R = 1;
        //最多需要二分20次
        for (int times = 0; times < 20; times++) {
            M = (L + R) / 2;
            build(root);
            for (int i = 1; i <= n; i++) last[i] = 0;
            bool ok = false;
            for (int i = 1; i <= n; i++) {
                //last表示data[i]上一次出现的位置
                updata(root, last[data[i]] + 1, i);
                //t表示能找到的size(l,r)+mid*l的最小值
                double t = query(root, i);
                if (t - M * (i + 1) <= 0) { ok = true; break; };
                last[data[i]] = i;
            }
            if (ok) R = M;
            else L = M;
        }
        printf("%f\n", (L + R) / 2);
    }
}
```