---
title: HDU4027 - Can you answer these queries? （线段树）
date: 2017-06-21 15:57:00
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=4027

-----------------------
# 题目分析：
区间开根号下取整，询问区间和。

-----------------------
# 解题过程：
注意，进行更新和询问的操作的时候要注意$x$和$y$的大小，这里被坑了，差点以为自己清奇的脑洞不对……

发现好多人都是转化为单点更新，自己比较耿直，写了好长的区间更新代码，也算是一题多解吧。

----------------
# 题目分析：
## 单点更新解法：
首先要注意到对于开根号操作，这里的输入数据大小不会超过$2^{64}$要不就没法玩了，这样的话，对于每个数顶多开$64$次根号。

这样暴力进行单点更新最多也就$O(64\times nlog(n))$，加上剪枝就可以过，对于每个区间，如果这个区间内的所有数都为$1$，那么这个区间开根号后区间和也没变化了，可以剪枝。

## 区间更新解法：
因为要进行区间更新，所以要用到lazy标记，但是用lazy标记要保证两个条件，一是标记可以叠加，二是打上标记后可以直接的更新区间维护的信息。

对于开根号操作，好像是不符合第二个条件，对于一个区间打上lazy标记后，不能直接计算出新的区间和。这里想到可以预处理前缀和，因为每个数最多也就开$64$次根号，预处理完对于开$1$到$64$次根号所有的前缀和，这里加下判断，如果所有数都为$1$的话，那么就不需要继续往下计算了。

有了前缀合，那么对于一段区间如果这段区间内所有数字开根号的次数都相同，那么我就可以用前缀和计算出区间和了。所以当一段区间内所有数的开根号次数都一样的话，就可以打lazy标记了。

然后注意更新和合并区间的过程，更新有可能会使一段区间内开根号次数不同，或变得相同，合并区间时要处理一下。

------------------------------------
# AC代码：
## 单点更新：
```cpp
#include <stdio.h>
#include <math.h>
#include <algorithm>
using namespace std;

#define lson root<<1
#define rson root<<1|1
#define MID int m = (l + r) / 2

typedef long long LL;

const int MAX = 112345;

struct Info {
    LL value;
}tree[MAX<<2];

LL data[MAX];

void build(int root, int l, int r) {
    if (l == r) {
        tree[root].value = data[l];
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m+1, r);
    tree[root].value = tree[lson].value + tree[rson].value;
}

void updata(int root, int l, int r, int ul, int ur) {
    if (r < ul || ur < l || tree[root].value <= (r-l+1))
        return;
    if (ul <= l && r <= ur && l == r) {
        tree[root].value = sqrt(tree[root].value);
        return;
    }
    MID;
    updata(lson, l, m, ul, ur);
    updata(rson, m+1, r, ul, ur);
    tree[root].value = tree[lson].value + tree[rson].value;
}

LL query(int root, int l, int r, int ul, int ur) {
    if (r < ul || ur < l)
        return 0;
    if (ul <= l && r <= ur) {
        return tree[root].value;
    }
    MID;
    return query(lson, l, m, ul, ur) + query(rson, m+1, r, ul, ur);
}

int main() {
    int n, m, cases = 0;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%lld", &data[i]);
        }
        build(1, 1, n);
        scanf("%d", &m);
        printf("Case #%d:\n", ++cases);
        while (m--) {
            int top, x, y;
            scanf("%d %d %d", &top, &x, &y);
            if (x > y)
                swap(x, y);
            if (top) {
                printf("%lld\n", query(1, 1, n, x, y));
            }
            else {
                updata(1, 1, n, x ,y);
            }
        }
        putchar('\n');
    }
}
```
## 区间更新：
```cpp
#include<stdio.h>
#include<math.h>
#include <algorithm>
using namespace std;

#define lson root<<1
#define rson root<<1|1
#define MID int m = (l+r)/2

const int MAX = 112345;
typedef long long LL;

LL pre[64][MAX];

int times;

struct Info {
    int lazy, l, r, num;
    bool ok;
    LL value;
    Info() {
        lazy = l = r = ok = num = value = 0;
    }
    void maintain(int v) {
        num += v;
        //如果开根号大于times次，那么和开times次相同
        if (num > times)
            num = times;
        value = pre[num][r] - pre[num][l-1];
    }
}tree[MAX*4];

void push_down(int root) {
    if (tree[root].lazy) {
        int v = tree[root].lazy;
        tree[lson].lazy += v;
        tree[rson].lazy += v;
        //打完lazy标记后重新计算下区间和
        tree[lson].maintain(v);
        tree[rson].maintain(v);
        tree[root].lazy = 0;
    }
}

Info operator + (const Info & a, const Info & b) {
    Info rst;
    rst.l = a.l;
    rst.r = b.r;
    //如果左右儿子开根号次数相同，并且区间内所有元素开根号次数都相同，那么父节点区间内开根号次数都相同
    rst.ok = (a.num == b.num) && a.ok && b.ok;
    if (rst.ok) {
        rst.num = a.num;
        rst.maintain(0);
    }
    return rst;
}

void updata(int root, int l, int r, int ul, int ur) {
    //如果区间内所有元素都为1，那么可以跳过更新
    if(r-l+1==tree[root].value) return;
    //如果区间在更新的区间内，并且区间内所有元素开根号次数都相同，那么可以打lazy标记
    if (ul <= l && r <= ur && tree[root].ok) {
        tree[root].lazy += 1;
        tree[root].maintain(1);
        return;
    }
    MID;
    push_down(root);
    if (ul <= m) updata(lson, l, m, ul, ur);
    if (m+1 <= ur) updata(rson, m+1, r, ul, ur);

    tree[root] = tree[lson] + tree[rson];
}

void build(int root, int l, int r) {
    tree[root].l = l;
    tree[root].r = r;
    tree[root].value = 0;
    tree[root].lazy = 0;
    tree[root].ok = 1;
    tree[root].num = 0;
    if (l == r) {
        tree[root].maintain(0);
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m+1, r);
    tree[root].value = tree[lson].value + tree[rson].value;
}

LL query(int root, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr && tree[root].ok) {
        return tree[root].value;
    }
    MID;
    push_down(root);
    LL sum = 0;
    if (ql <= m) sum += query(lson, l, m, ql, qr);
    if (m+1 <= qr) sum += query(rson, m+1, r, ql, qr);
    return sum;
}

int main() {
    int n, cases = 0;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%lld", &pre[0][i]);
            pre[1][i] = sqrt(pre[0][i]);
            pre[0][i] += pre[0][i-1];
        }

        //处理前缀和
        for (int i = 1; i <= 64; i++) {
            bool flag = 0;
            for (int j = 1; j <= n; j++) {
                if (pre[i][j] != 1)
                    flag = 1;
                pre[i+1][j] = sqrt(pre[i][j]);
                pre[i][j] += pre[i][j-1];
            }
            //如果所有元素均为1，就不用处理后面的前缀和了
            if (!flag) {
                //times为最多多少次，所有数字均变为1
                times = i;
                break;
            }
        }

        build(1, 1, n);
        int m;
        scanf("%d", &m);
        printf("Case #%d:\n", ++cases);
        while (m--) {
            int top, x, y;
            scanf("%d %d %d", &top, &x, &y);
            if (x > y)
                swap(x, y);
            if (top) {
                printf("%lld\n", query(1, 1, n, x, y));
            }
            else {
                updata(1, 1, n, x, y);
            }
        }
        putchar('\n');
    }
}
```