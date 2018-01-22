---
title: BZOJ1500 - 维修数列(Splay)
date: 2017-10-10 21:45:44
categories: [ACM, 数据结构, Splay]
tags:
---
# 题目链接：

http://www.lydsy.com/JudgeOnline/problem.php?id=1500

# 题目大意：

给出 N 个数字和 M 次操作。

分为下面六种操作：

![](http://www.lydsy.com/JudgeOnline/images/1500_1.jpg)

$M \le 2\times 10^4$，保证序列中的数字不会超过 $5 \times 10^5$，并且插入数字的总数不超过$4 \times 10^6$

# 解题过程：

调了一晚上才 A 掉，最后还是对照[金桔的代码](http://blog.csdn.net/a1s4z5/article/details/51890310)改的。

# 题目分析：

裸的 Splay 题，Splay的操作基本上都用上了，但是有好多坑点，下面列举一下：

- GET-SUM 有可能 y = 0，这是计算区间时有可能右区间大于左区间。
- 总共可能用到 $4 \times 10^6$ 个节点，这样会超内存，但是同时在序列的节点最多只有$5 \times 10^5$，所以要自己写内存回收。
- 求最大子列和需要维护的信息是不对称的，当节点翻转时，对应维护的信息也需要翻转。
- 当进行插入和删除操作的时候，需要维护一下根节点和插入到的父亲节点，主要是为了维护 size 这个值，因为 getid 需要用这个值二分，否则会 TLE。
- 区间修改时需要两个变量，一个是 lazy 值，另一个是判断是否进行了修改。


# AC代码：
```cpp
#include <cstdio>
#include <cstring>
#include <stack>

typedef long long ll;

using namespace std;

const int maxn = 552345 + 10;

int n, m;

struct Info {
    int size;
    ll sum;
    ll lma, rma, tma;

    Info(ll v = 0) {
        size = 1;
        sum = v;
        lma = rma = tma = v;
    }

    void addIt(ll v) {
        sum = v * size;
        lma = rma = tma = max(sum, v);
    }

    //翻转区间信息
    void revIt() {
        swap(lma, rma);
    }
};

//进行区间信息合并
Info operator+(const struct Info &a, const struct Info &b) {
    Info rst(a.sum + b.sum);
    rst.size = a.size + b.size;

    rst.lma = max(a.lma, a.sum + b.lma);
    rst.rma = max(b.rma, b.sum + a.rma);
    rst.tma = max(max(a.tma, b.tma), a.rma + b.lma);

    return rst;
}

struct Node {
    int son[2], fa;
    ll val, lazy;
    Info info;
    bool change;
    bool flip;

    int &l() { return son[0]; }

    int &r() { return son[1]; }

    Node(ll v = 0) {
        l() = r() = fa = -1;
        val = v;
        change = false;
        info = Info(v);
        lazy = 0;
        flip = false;
    }

    void maintain();

    void push_down();

    //翻转和修改操作
    void addIt(ll v) {
        val = v;
        lazy = v;
        change = true;
        info.addIt(v);
    }

    void revIt() {
        flip ^= 1;
        swap(l(), r());
        info.revIt();
    }
} node[maxn];

void Node::push_down() {
    if (change) {
        if (l() != -1) node[l()].addIt(lazy);
        if (r() != -1) node[r()].addIt(lazy);
        lazy = 0;
        change = false;
    }
    if (flip) {
        if (l() != -1) node[l()].revIt();
        if (r() != -1) node[r()].revIt();
        flip = false;
    }
}

void Node::maintain() {
    info = Info(val);
    if (l() != -1) info = node[l()].info + info;
    if (r() != -1) info = info + node[r()].info;
}

int ori(int st) {
    int fa = node[st].fa;
    if (fa == -1) return -1;
    return st == node[fa].r();
}

void setc(int st, int sn, int d) {
    if (st != -1) {
        node[st].son[d] = sn;
        node[st].maintain();
    }
    if (sn != -1) node[sn].fa = st;
}

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

int root;

#define f(x) (node[x].fa)

void splay(int x, int fa = -1) {
    while (f(x) != fa) {
        if (f(f(x)) == fa) zg(x);
        else {
            if (ori(x) == ori(f(x))) zg(f(x));
            else zg(x);
            zg(x);
        }
    }
    if (fa == -1) root = x;
}

int getid(int v, int st) {
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
    splay(r);
    splay(l, r);
    return node[l].r();
}


//进行插入和删除操作需要维护一下根节点和根节点的左儿子的区间信息
void segMaintain() {
    node[node[root].l()].maintain();
    node[root].maintain();
}

//进行内存回收
int head, tail;
int value[maxn], nxt[maxn];

int new_node(int v) {
    int rst = head;
    head = nxt[head];
    node[rst] = Node(v);
    return rst;
}

void recycle(int st) {
    if (st == -1) return;
    recycle(node[st].l());
    recycle(node[st].r());
    nxt[tail] = st;
    tail = st;
}

void del(int l, int r) {
    int pos = getseg(l, r);
    setc(node[pos].fa, -1, 1);
    recycle(pos);
    segMaintain();
}

int build(int l, int r) {
    int m = (l + r) >> 1;
    int st = new_node(value[m]);

    if (l < m) setc(st, build(l, m - 1), 0);
    if (m < r) setc(st, build(m + 1, r), 1);
    return st;
}

//初始化Splay
int build(int n) {
    head = 0;
    for (int i = 0; i < maxn; i++) {
        nxt[i] = i + 1;
    }
    tail = maxn - 1;
    nxt[tail] = -1;
    return build(0, n + 1);
}

void add(int l, int r, int v) {
    int pos = getseg(l, r);
    node[pos].addIt(v);
}

void insert(int pos, int p) {
    int l = pos;
    int r = pos + 1;
    l = getid(l + 1, root);
    r = getid(r + 1, root);
    splay(r);
    splay(l, r);
    setc(l, p, 1);
    segMaintain();
}

Info query(int l, int r) {
    return node[getseg(l, r)].info;
}

void flip(int l, int r) {
    int pos = getseg(l, r);
    node[pos].revIt();
}

int main() {
    char op[11];
    while (~scanf("%d %d", &n, &m)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", value + i);
        }
        value[n + 1] = 0;
        root = build(n);
        while (m--) {
            int x, y, z;
            scanf("%s", op);
            if (strcmp(op, "GET-SUM") == 0) {
                scanf("%d %d", &x, &y);
                if (y == 0) {
                    puts("0");
                    continue;
                }
                printf("%lld\n", query(x, x + y - 1).sum);
            } else if (strcmp(op, "MAX-SUM") == 0) {
                //因为插入了两个虚拟节点，所以要减二才是总共的节点数
                printf("%lld\n", query(1, node[root].info.size - 2).tma);
            } else if (strcmp(op, "INSERT") == 0) {
                scanf("%d %d", &x, &y);
                if (y == 0) continue;
                for (int i = 1; i <= y; i++) {
                    scanf("%d", value + i);
                }
                //用 build 根据刚刚输入的值生成一个 Splay 再与主 Splay 合并
                insert(x, build(1, y));
            } else if (strcmp(op, "DELETE") == 0) {
                scanf("%d %d", &x, &y);
                if (y == 0) continue;
                del(x, x + y - 1);
            } else if (strcmp(op, "REVERSE") == 0) {
                scanf("%d %d", &x, &y);
                if (y == 0) continue;
                flip(x, x + y - 1);
            } else if (strcmp(op, "MAKE-SAME") == 0) {
                scanf("%d %d %d", &x, &y, &z);
                if (y == 0) continue;
                add(x, x + y - 1, z);
            }
        }
    }
}
```