---
title: SDUTOJ3475 - 最后の汤圆
date: 2017-07-24 20:32:57
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3475.html

------------------------
# 题目大意：
给定一个序列$a_1, a_2 \dots a_n$，这个序列构成一个环，首先删掉第$k$个元素，然后从第$k$个元素开始，继续数$a_k$个元素删除掉下一个元素，求最后剩余的一个数的原始下标。


---------------------------------
# 解题过程：

类似约瑟夫环，之前你做过一个非常大的约瑟夫环直接套公式就好了，然后这一个仔细想了一下，删除每一个元素之后，下一个元素的位置不容易确定，感觉不能直接套公式那种，于是朴素的推了一下，还真的是线段树。

-----------------------------
# 题目分析：

刚开始每一个数的下标就对应他是第几个数比如$1,2,3,4,5$。
这里我如果删除第二个数，并且将第二个数及后面的数减一。
得到这个序列$1,1,2,3,4$，这里第一次出现的$1$，依然是第一个数。
假设我这时候再删除第一个数，那么就变成了$0,0,1,2,3$，依然符合上述的规律。

那么我们就可以这样模拟了，维护一个序列，初始化序列的值为对应的下标，每删除一个数，让这个数及后面的数的值减一。

然后找首个出现的时候用二分查找一下就可以，序列操作后也一直都是有序的，在线段树上也容易进行二分。

-----------------------------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

#define lson root<<1
#define rson root<<1|1
#define MID int m = (l+r)>>1

const int MAX = 500000+50;

struct Info {
    int v, lazy;
};

Info tree[MAX<<2];
int data[MAX];

Info operator + (const Info &a, const Info &b) {
    Info rst;
    rst.lazy = 0;
    //让根的值为右儿子的值，目的是让根节点的v是维护的这段区间的最大的
    rst.v = b.v;
    return rst;
}

void push_down(int root) {
    if (tree[root].lazy) {
        int lazy = tree[root].lazy;
        tree[lson].lazy += lazy;
        tree[rson].lazy += lazy;
        tree[lson].v -= lazy;
        tree[rson].v -= lazy;
        tree[root].lazy = 0;
    }
}

void build(int root, int l, int r) {
    tree[root].lazy = 0;
    if (l == r) {
        tree[root].v = l;
        return;
    }
    MID;
    build(lson, l, m);
    build(rson, m+1, r);
    tree[root] = tree[lson] + tree[rson];
}

void updata(int root, int l, int r, int ul, int ur) {
    if (ul > r || l > ur) return;
    //对区间进行减一
    if (ul <= l && r <= ur) {
        tree[root].lazy ++;
        tree[root].v --;
        return;
    }
    push_down(root);
    MID;
    updata(lson, l, m, ul, ur);
    updata(rson, m+1, r, ul, ur);
    tree[root] = tree[lson] + tree[rson];
}

int query(int root, int l, int r, int pos) {
    //如果到了叶子节点，那么到找到了答案
    if (l == r) return l;
    push_down(root);
    MID;
    //如果当前位置比左儿子区间内最大的还要大，那么只能去右儿子找
    if (pos > tree[lson].v) return query(rson, m+1, r, pos);
    else return query(lson, l, m, pos);
}

int main() {
    int n, k;
    while (~scanf("%d %d", &n, &k)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", data+i);
        }
        build(1, 1, n);

        //num表示为当前剩余的数字个数
        int num = n;
        //删除到只剩一个数
        while (num > 1) {
            //可能需要数的数字大于当前总共的，进行取模
            k = k % num;
            if (k == 0) k = num;

            //找到当前第k个数字的原始下标
            int index = query(1, 1, n, k);

            //将index及后面的值减一
            updata(1, 1, n, index, n);

            //接下来需要数从第k个数继续数data[index]个数
            //这等价于从第一个数开始数data[index]+k-1个数
            k = data[index] + k - 1;
            num--;
        }

        //最后查找一下剩余的最后一个数的下标并输出
        printf("%d\n", query(1, 1, n, 1));
    }
}
```