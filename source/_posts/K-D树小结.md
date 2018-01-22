---
title: K-D树小结
date: 2017-11-20 19:05:08
categories: [ACM, 数据结构, K-D树]
tags:
---
# 资料&感谢

[[Sengxian's Blog](https://blog.sengxian.com/)](https://blog.sengxian.com/algorithms/k-dimensional-tree)

[QSC算法讲堂](https://www.bilibili.com/video/av7039143/?from=search&seid=12824880607180236401)

# 介绍

K-D 树是 K 维树（k-dimensional tree）的缩写，可以用来寻找 K 维空间中距离一个点最近的若干个点，不过在 ACM 中一般用来处理平面的最近点对问题。

K-D 树实质上是一颗二叉树，所以可以做到插入删除点都是 $O(\log n)$，在建树的时候每次按一个维度排序后，取中间点把空间划分为两部分，在这个维度上小于中间点的放到左儿子，大于中间点的放到右儿子。

这里为了简单，首先从一维的 K-D 树入手。

## 一维的 K-D 树

![一维 K-D 树](http://chuantu.biz/t6/152/1511176656x-1566638157.png)

上图是一颗一维的 K-D，对于一维的情况，所有的点都是数轴上的点，那么这时候 
K-D 树就是一颗普通的二叉搜索树（Binary Search Tree）。

## 二维的 K-D 树

![二维 K-D 树](http://chuantu.biz/t6/152/1511176920x-1566638157.png)

这时候就是交替的按 x 维和 y 维排序进行划分的。上面就是对点集 `(2,3), (5,4), (9,6), (4,7), (8,1), (7,2)`的划分。

![](http://chuantu.biz/t6/152/1511177069x-1566638157.png)

上面的划分对应的树型结构就是这样的，对于第一层，左子树的点的 x 坐标都小于根节点，右子树的点的 x 坐标都大于根节点。对于第二层，左子树的点的 y 坐标都小于根节点，右子树的点的 y 坐标都大于根节点，以此类推。



# 构建

K-D 树的每个节点不仅要保存当前点的坐标信息，还要维护当前子树每个维度上的边界点。

这里 [Sengxian](https://blog.sengxian.com/algorithms/k-dimensional-tree) 巨巨写的非常好，就直接引用了。

> 在构造 1 维 BST 树时，一个 1 维数据根据其与树的根结点进行大小比较，来决定是划分到左子树还是右子树。
> 同理，我们也可以按照这样的方式，将一个 $k$ 维数据与 k-d树 的根结点进行比较，只不过不是对 $k$ 维数据进行整体的比较，而是选择某一个维度 $D_i$，然后比较两个数据在该维度 $D_i$ 上的大小关系，即每次选择一个维度 $D_i$ 来对 $k$ 维数据进行划分，相当于用一个垂直于该维度 $D_i$ 的超平面将 $k$ 维数据空间一分为二，平面一边的所有 $k$ 维数据在 $D_i$ 维度上的值小于平面另一边的所有 $k$ 维数据对应维度上的值。
> 也就是说，我们每选择一个维度进行如上的划分，就会将 $k$ 维数据空间划分为两个部分，如果我们继续分别对这两个子 $k$ 维空间进行如上的划分，又会得到新的子空间，对新的子空间又继续划分，重复以上过程直到每个子空间都不能再划分为止。以上就是构造 k-d树 的过程。
> 那么如果是二维特殊情况，就变得非常好理解了，通俗的来说就是通过过已有点的横线，竖线来划分二维平面。
> 上述过程中涉及到两个重要的问题：
>
> 1. 每次对子空间的划分时，怎样确定在哪个维度上进行划分？
> 2. 在某个维度上进行划分时，怎样确保在这一维度上的划分得到的两个子集合的数量尽量相等，即左子树和右子树中的结点个数尽量相等？
>
> 对于第一个问题，有很多种方法可以选择划分维度（axis-aligned splitting planes），所以有很多种创建 k-d树 的方法。 最典型的方法如下：
> 随着树的深度轮流选择维度来划分。例如，在二维空间中根节点以 x 轴划分，其子节点皆以 y 轴划分，其孙节点又以 x 轴划分，其曾孙节点则皆为 y 轴划分，依此类推。
> 另外的划分方法还有最大方差法（max invarince），在这里不做介绍。
>
> 而对于第二个问题，也是在 BST 中会遇到的一个问题。在 BST 中，我们是将数据的中位数作为根节点，然后再左右递归下去建树，这样可以得到一棵平衡的二叉搜索树。
> 同样，在 k-d树 中，若在维度 $D_i$ 上进行划分时，根节点就应该选择该维度 $D_i$ 上所有数据的中位数，这样递归子树的大小就基本相同了。



## K-D 树单个节点

```cpp
struct Node {
    int l, r;
    int id;
    //d 数组储存当前点，minn 和 maxn 表示当前节点维护的矩形的边界
    int d[DIM], minn[DIM], maxn[DIM];

    //对节点进行初始化
    inline void maintain() {
        for (int i = 0; i < DIM; i++) {
            minn[i] = maxn[i] = d[i];
        }
        l = r = 0;
    }
} tree[MAX];

//通过修改全局变量 D，实现按不同维度排序
bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}
```



## 建树

```cpp
int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    //将中间数放到 mid 位置，小于中间数的放左边，大于的放右边，不保证左右边有序，类似快排的一部分，复杂度O(N)
    nth_element(tree + l, tree + mid, tree + r + 1);
    //初始化节点信息
    tree[mid].maintain();
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    //维护子树信息
    pushUp(mid);
    re
```



## 插入

```cpp
void insert(int &o, int k, int now) {
    //如果当前节点为空，就直接将赋值
    if (o == 0) {
        o = k;
        return;
    }
    //否则根据 now 维的大小进行二分查找
    if (tree[k].d[now] < tree[o].d[now]) insert(tree[o].l, k, (now + 1) % DIM);
    else insert(tree[o].r, k, (now + 1) % DIM);
    //插入节点后更新信息
    pushUp(o);
}
```



## 查找

## 最近点

```cpp

//当前查找的点距离节点维护的矩阵的最近距离
int partionMin(int o, int k) {
    int rst = 0;
    for (int i = 0; i < DIM; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += tree[k].d[i] - tree[o].maxn[i];
        if (tree[k].d[i] < tree[o].minn[i]) rst += tree[o].minn[i] - tree[k].d[i];
    }
    return rst;
}

void query(int o, int k) {
    //通过当前节点储存的点更新答案
    int dm = abs(tree[o].d[0] - tree[k].d[0]) + abs(tree[o].d[1] - tree[k].d[1]);
    if (dm < ans) ans = dm;
    //计算左右子树距离当前点可能的最近的答案
    int dl = tree[o].l ? partionMin(tree[o].l, k) : INF;
    int dr = tree[o].r ? partionMin(tree[o].r, k) : INF;

    //通过搜索顺序进行剪枝
    if (dl < dr) {
        //如果最近可能的点都大于答案，那么不可能更新答案
        if (dl < ans) query(tree[o].l, k);
        if (dr < ans) query(tree[o].r, k);
    } else {
        if (dr < ans) query(tree[o].r, k);
        if (dl < ans) query(tree[o].l, k);
    }
}
```

初始化答案为无穷，从根节点开始，首先通过节点储存的点去更新答案。然后计算左右子树维护的矩形区域离当前查询的点的最近可能距离，然后首先搜距离近的，再搜远的，实际上就是一个剪枝。复杂度一般是$O(\log n)$，最差可能是$O(n \sqrt{n})$。

上面是曼哈顿距离，对于欧式几何距离用下面计算，也就是一个点到矩形的最近距离

```cpp
inline ll partionMin(int o, int k) {
    if (tree[o].minn[2] > tree[k].d[2]) return INF;
    ll rst = 0;
    for (int i = 0; i < 2; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += sqr(tree[k].d[i] - tree[o].maxn[i]);
        if (tree[k].d[i] < tree[o].minn[i]) rst += sqr(tree[o].minn[i] - tree[k].d[i]);
    }
    return rst;
}
```

扩展成求到某个点最近的 k 个点也比较容易，查询的时候维护一个最大堆就可以了，之前的剪枝，就是对对顶进行比较了，如果大于堆顶一定不可能更新答案。





# 题集

## BZOJ 2648

裸的 K-D 树题，求到某个点的最近曼哈顿距离。

```cpp
#include <bits/stdc++.h>
 
using namespace std;
 
typedef long long ll;
 
const int MAX = 500010;
const int DIM = 2;
const int INF = 0x3f3f3f3f;
 
struct Node {
    int l, r;
    int d[DIM], minn[DIM], maxn[DIM];
 
    inline void maintain() {
        for (int i = 0; i < DIM; i++) {
            minn[i] = maxn[i] = d[i];
        }
        l = r = 0;
    }
} tree[MAX * 2];
 
int D;
 
bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}
 
int root = 0, pos = 1;
 
inline void pushUp(int p) {
    int son[2] = {tree[p].l, tree[p].r};
    for (int i = 0; i < 2; i++) {
        if (!son[i]) continue;
        for (int j = 0; j < DIM; j++) {
            tree[p].maxn[j] = max(tree[son[i]].maxn[j], tree[p].maxn[j]);
            tree[p].minn[j] = min(tree[son[i]].minn[j], tree[p].minn[j]);
        }
    }
}
 
int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    nth_element(tree + l, tree + mid, tree + r + 1);
    tree[mid].maintain();
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    pushUp(mid);
    return mid;
}
 
int ans;
 
int partionMin(int o, int k) {
    int rst = 0;
    for (int i = 0; i < DIM; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += tree[k].d[i] - tree[o].maxn[i];
        if (tree[k].d[i] < tree[o].minn[i]) rst += tree[o].minn[i] - tree[k].d[i];
    }
    return rst;
}
 
void query(int o, int k) {
    int dm = abs(tree[o].d[0] - tree[k].d[0]) + abs(tree[o].d[1] - tree[k].d[1]);
    if (dm < ans) ans = dm;
    int dl = tree[o].l ? partionMin(tree[o].l, k) : INF;
    int dr = tree[o].r ? partionMin(tree[o].r, k) : INF;
    if (dl < dr) {
        if (dl < ans) query(tree[o].l, k);
        if (dr < ans) query(tree[o].r, k);
    } else {
        if (dr < ans) query(tree[o].r, k);
        if (dl < ans) query(tree[o].l, k);
    }
}
 
void insert(int &o, int k, int now) {
    if (o == 0) {
        o = k;
        return;
    }
    if (tree[k].d[now] < tree[o].d[now]) insert(tree[o].l, k, (now + 1) % DIM);
    else insert(tree[o].r, k, (now + 1) % DIM);
    pushUp(o);
}
 
int read() {
    int t;
    scanf("%d", &t);
    return t;
}
 
int main() {
    int n, m;
    n = read();
    m = read();
 
    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < DIM; j++) tree[i].d[j] = read();
    }
 
    root = build(1, n, 0);
 
    pos = n + 1;
    int op;
    while (m--) {
        op = read();
        for (int j = 0; j < DIM; j++) tree[pos].d[j] = read();
        if (op == 1) {
            tree[pos].maintain();
            insert(root, pos, 0);
            pos++;
        } else {
            ans = INF;
            query(root, pos);
            printf("%d\n", ans);
        }
    }
    return 0;
}
```



## BZOJ 1941

求每个点的最近点和最远的的距离差值最小的，用 K-D 树求每个点的最远点和最近点。

```cpp
#include <bits/stdc++.h>
 
using namespace std;
 
typedef long long ll;
 
const int MAX = 500010;
const int DIM = 2;
const int INF = 0x3f3f3f3f;
 
struct Node {
    int l, r;
    int d[DIM], minn[DIM], maxn[DIM];
 
    inline void maintain() {
        for (int i = 0; i < DIM; i++) {
            minn[i] = maxn[i] = d[i];
        }
        l = r = 0;
    }
} tree[MAX * 2];
 
int D;
 
bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}
 
int root = 0, pos = 1;
 
inline void pushUp(int p) {
    int son[2] = {tree[p].l, tree[p].r};
    for (int i = 0; i < 2; i++) {
        if (!son[i]) continue;
        for (int j = 0; j < DIM; j++) {
            tree[p].maxn[j] = max(tree[son[i]].maxn[j], tree[p].maxn[j]);
            tree[p].minn[j] = min(tree[son[i]].minn[j], tree[p].minn[j]);
        }
    }
}
 
int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    nth_element(tree + l, tree + mid, tree + r + 1);
    tree[mid].maintain();
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    pushUp(mid);
    return mid;
}
 
int ansMin, ansMax;
 
int partionMin(int o, int k) {
    int rst = 0;
    for (int i = 0; i < DIM; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += tree[k].d[i] - tree[o].maxn[i];
        if (tree[k].d[i] < tree[o].minn[i]) rst += tree[o].minn[i] - tree[k].d[i];
    }
    return rst;
}
 
int partionMax(int o, int k) {
    int rst = 0;
    for (int i = 0; i < DIM; i++) {
        rst += max(abs(tree[k].d[i] - tree[o].minn[i]), abs(tree[k].d[i] - tree[o].maxn[i]));
    }
    return rst;
}
 
void queryMin(int o, int k) {
    int dm = abs(tree[o].d[0] - tree[k].d[0]) + abs(tree[o].d[1] - tree[k].d[1]);
    if (o == k) dm = INF;
    if (dm < ansMin) ansMin = dm;
    int dl = tree[o].l ? partionMin(tree[o].l, k) : INF;
    int dr = tree[o].r ? partionMin(tree[o].r, k) : INF;
    if (dl < dr) {
        if (dl < ansMin) queryMin(tree[o].l, k);
        if (dr < ansMin) queryMin(tree[o].r, k);
    } else {
        if (dr < ansMin) queryMin(tree[o].r, k);
        if (dl < ansMin) queryMin(tree[o].l, k);
    }
}
 
void queryMax(int o, int k) {
    int dm = abs(tree[o].d[0] - tree[k].d[0]) + abs(tree[o].d[1] - tree[k].d[1]);
    if (dm > ansMax) ansMax = dm;
    int dl = tree[o].l ? partionMax(tree[o].l, k) : 0;
    int dr = tree[o].r ? partionMax(tree[o].r, k) : 0;
    if (dl > dr) {
        if (dl > ansMax) queryMax(tree[o].l, k);
        if (dr > ansMax) queryMax(tree[o].r, k);
    } else {
        if (dr > ansMax) queryMax(tree[o].r, k);
        if (dl > ansMax) queryMax(tree[o].l, k);
    }
}
 
int main() {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d %d", &tree[i].d[0], &tree[i].d[1]);
        }
 
        root = build(1, n, 0);
 
        int ans = INF;
 
        for (int i = 1; i <= n; i++) {
            ansMax = 0;
            ansMin = INF;
            queryMax(root, i);
            queryMin(root, i);
            ans = min(ans, ansMax - ansMin);
        }
        printf("%d\n", ans);
    }
    return 0;
}
```

## BZOJ 4066

给出一个$n \times n$的棋盘，每次对其中的一个点修改中，询问一个矩形，求矩形内点的和。

这里插入的点的数量有点多，可能导致树的形态太差，所以插入点树每过 10000 就暴力重构一下。

```cpp
#include <bits/stdc++.h>
 
using namespace std;
 
typedef long long ll;
 
const int MAX = 200010;
const int DIM = 2;
const int INF = 0x3f3f3f3f;
 
struct Node {
    int l, r;
    int d[DIM], minn[DIM], maxn[DIM];
    int sum, v;
 
    inline void maintain() {
        for (int i = 0; i < DIM; i++) {
            minn[i] = maxn[i] = d[i];
        }
        sum = v;
        l = r = 0;
    }
} tree[MAX * 2];
 
int D;
 
bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}
 
int root = 0, pos = 1;
 
inline void pushUp(int p) {
    int son[2] = {tree[p].l, tree[p].r};
    for (int i = 0; i < 2; i++) {
        if (!son[i]) continue;
        for (int j = 0; j < DIM; j++) {
            tree[p].maxn[j] = max(tree[son[i]].maxn[j], tree[p].maxn[j]);
            tree[p].minn[j] = min(tree[son[i]].minn[j], tree[p].minn[j]);
        }
    }
    tree[p].sum = tree[son[0]].sum + tree[p].v + tree[son[1]].sum;
}
 
int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    nth_element(tree + l, tree + mid, tree + r + 1);
    tree[mid].maintain();
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    pushUp(mid);
    return mid;
}
 
int ans;
int xl, xr, yl, yr;
 
void query(int o) {
    if (xr < tree[o].minn[0] || tree[o].maxn[0] < xl  ||
        yr < tree[o].minn[1] || tree[o].maxn[1] < yl)
        return;
 
    if (xl <= tree[o].minn[0] && tree[o].maxn[0] <= xr &&
        yl <= tree[o].minn[1] && tree[o].maxn[1] <= yr) {
        ans += tree[o].sum;
        return;
    }
 
    if (xl <= tree[o].d[0] && tree[o].d[0] <= xr &&
        yl <= tree[o].d[1] && tree[o].d[1] <= yr)
        ans += tree[o].v;
 
    if (tree[o].l) query(tree[o].l);
    if (tree[o].r) query(tree[o].r);
}
 
void insert(int &o, int k, int now) {
    if (o == 0) {
        o = k;
        return;
    }
    if (tree[k].d[now] < tree[o].d[now]) insert(tree[o].l, k, (now + 1) % DIM);
    else insert(tree[o].r, k, (now + 1) % DIM);
    pushUp(o);
}
 
int read(bool type = true) {
    int t;
    scanf("%d", &t);
    if (type) t ^= ans;
    return t;
}
 
int main() {
    int n;
    scanf("%d", &n);
    int op;
    while (true) {
        op = read(0);
        if (op == 3) break;
        if (op == 1) {
            for (int j = 0; j < DIM; j++) tree[pos].d[j] = read();
            tree[pos].v = read();
            tree[pos].maintain();
            insert(root, pos, 0);
            pos++;
            if (pos % 10000 == 0) root = build(1, pos - 1, 0);
        } else {
            xl = read();
            yl = read();
            xr = read();
            yr = read();
 
            ans = 0;
            query(root);
            printf("%d\n", ans);
        }
    }
    return 0;
}

```

## HDU 5992

三维 K-D 树，求二维平面是距离某个点最近的点，并且第三维不超过某个值。

```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

const int MAX = 200010;
const int DIM = 3;
const ll INF = 0x3f3f3f3f3f3f3f3f;

struct Node {
    int l, r;
    int id;
    int d[DIM], minn[DIM], maxn[DIM];
} tree[MAX];

int D;

bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}

int root;

inline void pushUp(int p) {
    int son[2] = {tree[p].l, tree[p].r};
    for (int i = 0; i < 2; i++) {
        if (!son[i]) continue;
        for (int j = 0; j < DIM; j++) {
            tree[p].maxn[j] = max(tree[son[i]].maxn[j], tree[p].maxn[j]);
            tree[p].minn[j] = min(tree[son[i]].minn[j], tree[p].minn[j]);
        }
    }
}

int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    nth_element(tree + l, tree + mid, tree + r + 1);
    for (int i = 0; i < 3; i++) tree[mid].maxn[i] = tree[mid].minn[i] = tree[mid].d[i];
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    pushUp(mid);
    return mid;
}

inline ll sqr(const ll &x) {
    return x * x;
}

inline ll dis(const Node &a, const Node &b) {
    return sqr((ll) a.d[0] - (ll) b.d[0]) + sqr((ll) a.d[1] - (ll) b.d[1]);
}

inline ll partionMin(int o, int k) {
    if (tree[o].minn[2] > tree[k].d[2]) return INF;
    ll rst = 0;
    for (int i = 0; i < 2; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += sqr(tree[k].d[i] - tree[o].maxn[i]);
        if (tree[k].d[i] < tree[o].minn[i]) rst += sqr(tree[o].minn[i] - tree[k].d[i]);
    }
    return rst;
}

ll ans1;
int ans2, ansNode;

void query(int o, int k) {
    ll dm = dis(tree[o], tree[k]);
    if (tree[o].d[2] <= tree[k].d[2] && (dm < ans1 || dm == ans1 && tree[o].id < ans2)) {
        ans1 = dm;
        ans2 = tree[o].id;
        ansNode = o;
    }
    ll dl = tree[o].l ? partionMin(tree[o].l, k) : INF;
    ll dr = tree[o].r ? partionMin(tree[o].r, k) : INF;
    if (dl < dr) {
        if (dl <= ans1) query(tree[o].l, k);
        if (dr <= ans1) query(tree[o].r, k);
    } else {
        if (dr <= ans1) query(tree[o].r, k);
        if (dl <= ans1) query(tree[o].l, k);
    }
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, m;
        scanf("%d %d", &n, &m);
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < DIM; j++) scanf("%d", &tree[i].d[j]);
            tree[i].l = tree[i].r = 0;
            tree[i].id = i;
        }
        root = build(1, n, 0);
        while (m--) {
            for (int j = 0; j < DIM; j++) scanf("%d", &tree[n + 1].d[j]);
            ans1 = INF;
            query(root, n + 1);
            printf("%d %d %d\n", tree[ansNode].d[0], tree[ansNode].d[1], tree[ansNode].d[2]);
        }
    }
}
```

## HDU 4347

求 k 维的最近 m 个点。

```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long ll;

const int MAX = 50000 + 100;
const int MAXDIM = 5;
const ll INF = 0x3f3f3f3f3f3f3f3f;

int DIM;

struct Node {
    int l, r;
    int id;
    int d[MAXDIM], minn[MAXDIM], maxn[MAXDIM];
    inline void maintain() {
        for (int i = 0; i < DIM; i++) {
            minn[i] = maxn[i] = d[i];
        }
        l = r = 0;
    }
} tree[MAX];

int D;
priority_queue<pair<ll, int> > q;

bool operator<(const Node &a, const Node &b) {
    return a.d[D] < b.d[D];
}

int root;

inline void pushUp(int p) {
    int son[2] = {tree[p].l, tree[p].r};
    for (int i = 0; i < 2; i++) {
        if (!son[i]) continue;
        for (int j = 0; j < DIM; j++) {
            tree[p].maxn[j] = max(tree[son[i]].maxn[j], tree[p].maxn[j]);
            tree[p].minn[j] = min(tree[son[i]].minn[j], tree[p].minn[j]);
        }
    }
}

int build(int l, int r, int now) {
    int mid = (l + r) >> 1;
    D = now;
    nth_element(tree + l, tree + mid, tree + r + 1);
    tree[mid].maintain();
    if (l < mid) tree[mid].l = build(l, mid - 1, (now + 1) % DIM);
    if (mid < r) tree[mid].r = build(mid + 1, r, (now + 1) % DIM);
    pushUp(mid);
    return mid;
}

inline ll sqr(const ll &x) {
    return x * x;
}

inline ll dis(const Node &a, const Node &b) {
    ll rst = 0;
    for (int i = 0; i < DIM; i++) {
        rst += sqr((ll) a.d[i] - (ll) b.d[i]);
    }
    return  rst;
}

inline ll partionMin(int o, int k) {
    ll rst = 0;
    for (int i = 0; i < DIM; i++) {
        if (tree[k].d[i] > tree[o].maxn[i]) rst += sqr(tree[k].d[i] - tree[o].maxn[i]);
        if (tree[k].d[i] < tree[o].minn[i]) rst += sqr(tree[o].minn[i] - tree[k].d[i]);
    }
    return rst;
}

void query(int o, int k) {
    ll dm = dis(tree[o], tree[k]);

    if (dm < q.top().first) {
        q.pop();
        q.push(make_pair(dm, o));
    }

    ll dl = tree[o].l ? partionMin(tree[o].l, k) : INF;
    ll dr = tree[o].r ? partionMin(tree[o].r, k) : INF;
    if (dl < dr) {
        if (dl <= q.top().first) query(tree[o].l, k);
        if (dr <= q.top().first) query(tree[o].r, k);
    } else {
        if (dr <= q.top().first) query(tree[o].r, k);
        if (dl <= q.top().first) query(tree[o].l, k);
    }
}

int main() {
    int n;
    while (~scanf("%d %d", &n, &DIM)) {
        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < DIM; j++) {
                scanf("%d", &tree[i].d[j]);
            }
        }
        root = build(1, n, 0);
        int t, m;
        scanf("%d", &t);
        while (t--) {
            for (int i = 0; i < DIM; i++) {
                scanf("%d", &tree[n + 1].d[i]);
            }
            scanf("%d", &m);
            while (!q.empty()) q.pop();
            for (int i = 0; i < m; i++) q.push(make_pair(INF, -1));
            query(root, n + 1);
            vector<int> ans;
            while (!q.empty()) {
                ans.push_back(q.top().second);
                q.pop();
            }
            printf("the closest %d points are:\n", m);
            for (int i = ans.size() - 1; i >= 0; i--) {
                for (int j = 0; j < DIM; j++) {
                    printf("%d%c", tree[ans[i]].d[j], j == DIM - 1 ? '\n' : ' ');
                }
            }
        }
    }
}
```

