---
title: AC自动机（模板）
date: 2017-09-25 14:13:33
categories: [ACM, 字符串, AC自动机]
tags:
---


# 感谢&资料：

主要参考紫书的写法，其他的写法都用到了指针，不太喜欢指针...

因为紫书没有完整的代码，然后参考了一下 [dalao 的博客](http://www.acyume.com/archives/19)。



# 简介：

AC自动机主要解决的是字符串匹配问题，不同于 KMP 的是，AC 自动机可以进行多个模式串的匹配。KMP 的失配指针是一个线性的数组，但是 AC 自动机是多个模式串匹配，所以适配指针是一个树型的结构。



# 代码：

```cpp
#include <bits/stdc++.h>

using namespace std;

//字典树的最大节点个数
const int MAX = 250001;
const int N = 1000010;
//有多少个不同的字符
const int SIGMA_SIZE = 26;

//字典树的节点
int ch[MAX][SIGMA_SIZE];
//当前节点是否为一个模式串的结尾， 当前节点的上一个模式串结尾， fail指针
int val[MAX], last[MAX], f[MAX], sz;
int ANS;

void init() {
    sz = 1;
    memset(ch, 0, sizeof(ch));
    memset(val, 0, sizeof(val));
    memset(f, 0, sizeof(f));
    memset(last, 0, sizeof(last));
}

int idx(char c) {
    return c - 'a';
}

//更新答案
void add(int u) {
    while (u) {
        ANS += val[u];
        val[u] = 0;
        u = last[u];
    }
}

//添加模式串
void Creat(char *s) {
    int u = 0, len = strlen(s);
    for (int i = 0; i < len; i++) {
        int c = idx(s[i]);
        if (!ch[u][c]) ch[u][c] = sz++;
        u = ch[u][c];
    }
    val[u]++;
}


//获得fail指针
void getFail() {
    queue<int> q;
    for (int i = 0; i < SIGMA_SIZE; i++)
        if (ch[0][i]) q.push(ch[0][i]);

    while (!q.empty()) {
        int r = q.front(); q.pop();
        for (int c = 0; c < SIGMA_SIZE; c++) {
            int u = ch[r][c];
            if (!u) continue;
            q.push(u);
            int v = f[r];
            //和kmp相似，和根据父亲的fail指针获得当前的
            while (v && ch[v][c] == 0) v = f[v];
            f[u] = ch[v][c];
            //更新last
            last[u] = val[f[u]] ? f[u] : last[f[u]];
        }
    }
}

//进行匹配
void find(char * T) {
    int len = strlen(T), j = 0;
    for (int i = 0; i < len; i++) {
        int c = idx(T[i]);
        while (j && ch[j][c] == 0) j = f[j];
        j = ch[j][c];
        //如果当前是模式串的结尾，那么更新答案
        //else if 里是处理一个模式串包含另一个模式串的情况
        if (val[j]) add(j);
        else if (last[j]) add(last[j]);
    }
}

char str[N];

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        init();
        int n;
        scanf("%d", &n);
        while (n--) {
            scanf("%s", str);
            Creat(str);
        }
        getFail();
        scanf("%s", str);
        ANS = 0;
        find(str);
        printf("%d\n", ANS);
    }
}
```

