---
title: HDU3487 - Play with Chain(Splay + 模板详解)
date: 2017-10-09 21:36:56
categories: [ACM, 数据结构, Splay]
tags:
---
# 题目链接：

[https://vjudge.net/problem/HDU-3487](https://vjudge.net/problem/HDU-3487)



# 题目大意：

给出一个 1 ~ n 的序列，有 m 次操作，分为以下两种：

+ CUT a, b, c 将区间 a ~ b 剪下来，放到剩下的序列中第 c 个元素后面。
+ FLIP a, b 将区间 a ~ b 翻转。

输出执行完全部操作后的序列。

$1 \le n, m \le 3 \times 100000$

# 解题过程：

模板题，对着板子敲的，理解了下 Splay 的细节。



# 题目分析：

区间翻转和区间删除插入都是 Splay 树的经典操作，然后会 Splay 后这就是一个模板题。




# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;


//用来处理区间询问，每个节点维护的是一个子树的信息
//Splay可以将一段连续区间内的节点放到一颗子树内，所以这样可以维护一段区间的信息
struct Info {
    int size;
    int ma;

    Info() {};

    Info(int x) {
        size = 1;
        ma = x;
    }

    void addIt(int x) {
        ma += x;
    }
};

//区间（子树）信息的合并
Info operator+(const Info &l, const Info &r) {
    Info ret;
    ret.size = l.size + r.size;
    ret.ma = max(l.ma, r.ma);
    return ret;
}

const int maxn = 3 * 100000 + 10;

int root;

//Splay的节点
struct Node {
    //son[0]是左儿子，son[1]是右儿子
    int son[2], fa;
    int val, lazy;
    bool flp;
    Info info;

    int &l() { return son[0]; }

    int &r() { return son[1]; }

    Node(int v = 0) {
        l() = r() = fa = -1;
        val = v;
        info = Info(v);
        lazy = 0;
        flp = false;
    }

    //修改Splay上的节点后，也需要对 info 的信息进行维护
    void addIt(int v) {
        val += v;
        lazy += v;
        info.addIt(v);
    }

    void maintain();

    void push_down();
} node[maxn];


//进行 pushdown 操作，类似线段树
void Node::push_down() {
    if (lazy != 0) {
        if (l() != -1) node[l()].addIt(lazy);
        if (r() != -1) node[r()].addIt(lazy);
        lazy = 0;
    }
    if (flp) {
        swap(l(), r());
        if (l() != -1) node[l()].flp ^= 1;
        if (r() != -1) node[r()].flp ^= 1;
        flp = false;
    }
}

//Splay 进行旋转操作时，子树发生了改变，需要重新维护区间（子树）信息
void Node::maintain() {
    info = Info(val);
    if (l() != -1) info = node[l()].info + info;
    if (r() != -1) info = info + node[r()].info;
}


//查询当前节点是父亲的左儿子还是右儿子，左儿子返回0，右儿子返回1，如果无父亲返回-1
int ori(int st) {
    int fa = node[st].fa;
    if (fa == -1) return -1;
    return st == node[fa].r();
}


//把 sn 变成 st 的儿子节点，如果 d 是 0 是左儿子，否则是右儿子
//这里子树发生了改变，需要重新维护 info 信息
void setc(int st, int sn, int d) {
    if (st != -1) {
        node[st].son[d] = sn;
        node[st].maintain();
    }
    if (sn != -1) node[sn].fa = st;
}

//进行旋转操作，这里需要自己画图理解一下
void zg(int x) {
    int st = node[x].fa, p = -1;
    node[st].push_down();
    node[x].push_down();

    int d = ori(x), dst = ori(st);
    if (st != -1) p = node[st].fa;
    setc(st, node[x].son[d ^ 1], d);
    setc(x, st, d ^ 1);
    setc(p, x, dst);
}

#define f(x) (node[x].fa)

//将 x 旋转成 fa 的儿子，如果将 x 旋转成 根节点的话则不填 fa
void splay(int x, int fa = -1) {
    //循环直到 x 是 fa 的儿子
    while (f(x) != fa) {
        //如果 fa 是 x 的爷爷，那么只需要一次旋转
        if (f(f(x)) == fa) zg(x);
        else {
            //双旋！

            //说明进行 zig zig 或者 zag zag 旋转
            if (ori(x) == ori(f(x))) zg(f(x));
            else zg(x);
            zg(x);
        }
    }
    //更新根节点
    if (fa == -1) root = x;
}

int value[maxn];
int pos;

//要保证 value 有序，类似线段树建树，这样树高是 log(n) 的
int build(int l, int r) {
    int st = pos++;
    int m = (l + r) >> 1;
    node[st] = Node(value[m]);
    if (l < m) setc(st, build(l, m - 1), 0);
    if (m < r) setc(st, build(m + 1, r), 1);
    return st;
}

int build(int n) {
    pos = 0;
    //添加 0 和 n + 1 两个虚拟节点，方便 cut 操作
    return build(0, n + 1);
}

//获得以 st 为根节点，中序遍历的第 v 个节点
int getid(int v, int st) {
    //在树上进行二分
    node[st].push_down();
    int l = node[st].l();
    int lsize = 1 + (l == -1 ? 0 : node[l].info.size);
    if (v == lsize) return st;
    int d = v > lsize;
    if (d) v -= lsize;
    return getid(v, node[st].son[d]);
}

int getseg(int l, int r) {
    l--, r++;
    l = getid(l + 1, root), r = getid(r + 1, root);
    //现在 r+1 是 l-1 的父亲，那么 l-r 这一段子树肯定是 l-1 的右儿子
    splay(r);
    splay(l, r);
    return node[l].r();
}

void flip(int l, int r) {
    int pos = getseg(l, r);
    node[pos].flp ^= 1;
}

void cut(int l, int r, int idx) {
    //切下来 l - r 这段区间
    int rootson1 = getseg(l, r);
    int father = node[rootson1].fa;
    setc(father, -1, 1);
    l = idx, r = idx + 1;
    //因为这里是虚拟节点，所以要多加一个 1
    l = getid(l + 1, root);
    r = getid(r + 1, root);
    //将 idx+1 成为 idx 的父亲，那么上面切下来的区间放到idx的右边即可
    splay(r);
    splay(l, r);
    setc(l, rootson1, 1);
}

int n, m;
int ans[maxn], cnt;

//中序遍历
void dfs(int now) {
    node[now].push_down();
    if (node[now].son[0] != -1) dfs(node[now].son[0]);
    ans[cnt++] = node[now].val;
    if (node[now].son[1] != -1) dfs(node[now].son[1]);
}

int main() {
    char op[10];
    int L, R, idx;
    while (~scanf("%d %d", &n, &m)) {
        if (n == -1 && m == -1) break;
        for (int i = 1; i <= n; i++) value[i] = i;
        root = build(n);
        while (m--) {
            scanf("%s %d %d", op, &L, &R);
            if (op[0] == 'F') flip(L, R);
            else {
                scanf("%d", &idx);
                cut(L, R, idx);
            }
        }
        cnt = 0;
        memset(ans, 0, sizeof(ans));
        dfs(root);
        for (int i = 1; i <= n; i++) printf("%d%c", ans[i], i == n ? '\n' : ' ');
    }
}
```