---
title: 2-SAT问题（模板）
date: 2017-10-18 09:32:46
categories: [ACM, 图论, 2-SAT]
tags:
---
# 简介

2-SAT （[2-satisfiability](https://en.wikipedia.org/wiki/2-satisfiability)）是描述一个这样的问题，有 n 个 bool 变量 $x_i$，并且有 m 个需要满足的条件，比如： "$x_1$为真或$x_2$为假"，“ $x_1$ 为真或$x_2$为真”之类的条件，这里”或“是指两个条件中至少有一个为真。SAT的问题是确定这 n 个变量的值，使得满足所有的条件。

# 解法

以下主要参考[Sengxian's Blog](https://blog.sengxian.com/algorithms/2-sat)和刘汝佳的白书。

有一个比较容易理解的解法，首先将每一个变量当成两个图中的顶点，比如 $x_i$ 拆成 $2i$ 和 $2i + 1$ 两个节点，分表表示 $x_i$ 为假和真。比如标记了 $2i + 1$ 这个节点表示 $x_i$ 这个变量为真，如果标记了 $2i $ 则表示 $x_i$ 为假。

对于 "$x_i$ 为真或 $x_j$ 为假"这个条件，我们添加一条 $2i$ 到 $2j$ 的边，表示如果 $x_i$ 为假的话，那么要使得条件成立 $x_j$ 一定要为假。另外同理也要添加一条 $2j + 1$ 到 $2i + 1$的边。注意上面的都是有向边，这里的边可以当做逻辑上的推导出的意思。

这样根据上面建完图后，接下来逐一考虑没有标记的变量，设为 $x_i$。我们先假定它为假，然后标记节点 $2i$，并且沿着有向边标记所有能标记的节点。如果标记过程中发现某个变量所对应的两个节点都被标记了，则 " $x_i$ 为假" 这个假定不成立，需要改成 " $x_i$ 为真"，然后退回到标记 " $x_i$ 为假" 之前的状态，重新操作。注意，如果当前考虑的变量不管是真是假都会引起矛盾，可以证明整个 2-SAT 问题无解（即使调整以前赋值的其他变量都没用），所以这个算法是没有回溯过程的，这样最差的复杂度是 $O(N \cdot M)$。

其实对于 2-SAT 问题还 $O(M)$ 的算法，不过对于 2-SAT 问题一般是考的建图方式，不卡时间，这里给出几个链接：

+  [http://blog.csdn.net/hqd_acm/article/details/5881655](http://blog.csdn.net/hqd_acm/article/details/5881655)
+ [http://www.cppblog.com/MatoNo1/archive/2015/12/29/150766.html](http://www.cppblog.com/MatoNo1/archive/2015/12/29/150766.html)



```cpp
struct TwoSat {
    static const int MAX_NODE = 1000;
    vector<int> G[MAX_NODE];
    int n, stk[MAX_NODE], sz;
    bool mark[MAX_NODE];

    void init(int _n) {
        n = _n;
        for (int i = 0; i < n * 2; ++i) G[i].clear();
        memset(mark, 0, sizeof(mark));
    }

    void addClause(int x, int xVal, int y, int yVal) {
        x = x * 2 + xVal, y = y * 2 + yVal;
        G[x ^ 1].push_back(y);
        G[y ^ 1].push_back(x);
    }

    bool dfs(int x) {
        if (mark[x ^ 1]) return false;
        if (mark[x]) return true;
        stk[sz++] = x;
        mark[x] = true;
        for (int i = 0; i < (int)G[x].size(); ++i)
            if (!dfs(G[x][i])) return false;
        return true;
    }

    bool solve() {
        for (int i = 0; i < n * 2; i += 2)
            if (!mark[i] && !mark[i ^ 1]) {
                sz = 0;
                if (!dfs(i)) {
                    while (sz > 0) mark[stk[--sz]] = false;
                    if (!dfs(i ^ 1)) return false;
                }
            }

        return true;
    }
};
```

# 扩展

上面描述的条件都只是 “或”，即是两个之中有一个成立，这里可以通过多个“或”条件的组合产生其他的逻辑条件。



| 条件              | 转化                                       | 实现                                       |
| --------------- | ---------------------------------------- | ---------------------------------------- |
| $a=b$           | $a \vee \lnot b \bigwedge \lnot a \vee b  $ | add_clause(a, 1, b, 0); add_clause(a, 0, b, 1); |
| $a \neq b$      | $a \vee b \bigwedge \lnot a \vee \lnot b$ | add_clause(a, 0, b, 0); add_clause(a, 1, b, 1); |
| $a = b = true$  | $a \vee \lnot b \bigwedge \lnot a \vee b  \bigwedge a \vee b$ | add_clause(a, 1, b, 1); add_clause(a, 1, b, 0); add_clause(a, 0, b, 1); |
| $a = b = false$ | $a \vee \lnot b \bigwedge \lnot a \vee b  \bigwedge \lnot a \vee  \lnot b$ | add_clause(a, 0, b, 0); add_clause(a, 1, b, 0); add_clause(a, 0, b, 1); |
