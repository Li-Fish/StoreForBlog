---
title: HDU4288 - Coder（线段树）
date: 2017-08-08 10:43:50
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=4288

--------------------
# 题目大意：
给出一个有小到大排序的集合，初始化为空。

三种操作：
1. 插入一个数。
2. 删除一个数。
3. 询问 $\sum_{1 \le i \le k} a_i, \text{where} i \mod 5 = 3$

-------------------
# 解题过程：
当初好多人都是暴力水过去的，然后刚开始看错了题意，然后试了好多次都不行...
然后重新写暴力TLE，最后没时间也放弃了，比赛后就直接用线段树去补了。

--------------------
# 题目分析：
由于集合是有序的，先离散化输入的数据，用线段树去维护这个列表。

用线段树去维护两个值，一个是线段树当前区间内已经插入的数字个数，一个是区间内下标%5分别等于0~4的已插入的元素的和。

然后合并的时候父节点的`sum[i] =  sum1[i] + sum2[((i-cnt1)%5+5)%5]`，cnt1为左儿子插入的元素个数。

----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

const int MAX = 112345;

#define lson o<<1
#define rson o<<1|1
#define MID int m = (l + r) >> 1

struct Node {
    ll sum[5];
    int cnt;
} tree[MAX<<2];

void merge(int o) {
    //进行合并，可以理解成，左儿子插入过了几个元素，右儿子要找的位置就要向左移几次
    for (int i = 0; i < 5; i++) {
        tree[o].sum[i] = tree[lson].sum[i]
                         + tree[rson].sum[((i - tree[lson].cnt) % 5 + 5) % 5];
    }
}

void build(int o, int l, int r) {
    tree[o].cnt = 0;
    memset(tree[o].sum, 0, sizeof(tree[o].sum));
    if (l == r) return;
    MID;
    build(lson, l, m);
    build(rson, m + 1, r);
}

void updata(int o, int l, int r, int pos, int v) {
    if (r < pos || pos < l) return;
    //记录插入的元素个数
    tree[o].cnt += v > 0 ? 1 : -1;
    if (l == r) {
        //区间长度为1时，只有下标%5=1的元素
        tree[o].sum[1] += v;
        return;
    }
    MID;
    updata(lson, l, m, pos, v);
    updata(rson, m + 1, r, pos, v);
    merge(o);
}

int n;
char op[MAX][112];
int tmp[MAX], data[MAX];

int main() {
    while (~scanf("%d", &n)) {
        int num = 0;
        for (int i = 0; i < n; i++) {
            scanf("%s", op[i]);
            if (op[i][0] != 's') {
                scanf("%d", &data[i]);
                tmp[num++] = data[i];
            }
        }
        //对插入的元素进行离散化
        sort(tmp, tmp + num);
        num = unique(tmp, tmp + num) - tmp;
        build(1, 1, num);
        for (int i = 0; i < n; i++) {
            int pos = lower_bound(tmp, tmp + num, data[i]) - tmp + 1;
            if (op[i][0] == 'a') updata(1, 1, num, pos, data[i]);
            if (op[i][0] == 'd') updata(1, 1, num, pos, -data[i]);
            //每次询问整个区间下标%5=3的和
            if (op[i][0] == 's') printf("%lld\n", tree[1].sum[3]);
        }
    }
}

```