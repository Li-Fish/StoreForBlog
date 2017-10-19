---
title: POJ2155 - Matrix （二维树状数组）
date: 2017-04-09 17:00:47
categories: [ACM, 数据结构, 线段树]
tags:
---
# 题目链接：
http://poj.org/problem?id=2155

----------------------
# 题目大意：
 给定一个矩阵，初始化为0，现在可以进行两种操作，一种是查询某个点的值是 0 还是 1。另一种是让这个矩阵的一个子矩阵内的值取反。

-----------------------------
# 解题过程：
 省赛选拔赛的题，太难了直接没看………
 后来补起来，有模板还是挺容易的。

------------------------------
# 题目分析：
+ 首先这题虽然看起来像是一个区间修改，单点查询的题，但是可以转化成单点修改，查询区间和。
	+   首先考虑一维的情况，我要一段区间取反，假设是 [l, r]。那么我只需要`book[l]+1`，`book[r+1]+1`，假设查询 k 的时候，只需要查询前 k 的和 mod 2 的结果即可。
	+ 然后这种方法可以推广到二维，不过这里要用一下容斥原理。假设修改的子矩阵左上角和右下角分别为 x1 y1 x2 y2，首先`book[x1][y1]+1`, `book[x2][y2]+1`，不过这时要 `book[x1][y2+1]+1`, `book[x2+1][y1]+1`。
	
	+ ![这里写图片描述](http://img.blog.csdn.net/20170409165447611?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
	
	+ 这里红色的就是上面需要标记的点，标记后表示以这个点为左下角的子矩阵内的点全部取反一次。这里绿色代表取反一次，紫色是取反了两次，黄色是取反了四次。最后实际进行取反操作的就是要求取反的子矩阵内的点。
+ 注意这里的前缀和用二位树状数组维护，进行单点更新，单点查询。

# AC代码：
```cpp
#include<cstdio>
#include<cstring>
using namespace std;

const int MAX = 1123;
int data[MAX][MAX], n;

int lowbit(int x) {
    return x&-x;
}

void Add(int x, int y, int w) {
    for (int i = x; i <= n; i += lowbit(i)) {
        for (int j = y; j <= n; j += lowbit(j)) {
            data[i][j] += w;
        }
    }
}

int Sum(int x, int y) {
    int ans = 0;
    for (int i = x; i > 0; i -= lowbit(i)) {
        for (int j = y; j > 0; j -= lowbit(j)) {
            ans += data[i][j];
        }
    }
    return ans;
}

int main() {
    char str[10];
    int T;
    scanf("%d", &T);
    while (T--) {
        int k;
        scanf("%d %d", &n, &k);
        memset(data, 0, sizeof(data));
        while (k--) {
            scanf(" %s", str);
            if (str[0] == 'C') {
                int x1, x2, y1, y2;
                scanf("%d %d %d %d", &x1, &y1, &x2, &y2);
                Add(x1, y1, 1);
                Add(x2+1, y1, 1);
                Add(x1, y2+1, 1);
                Add(x2+1, y2+1, 1);
            }
            else {
                int x, y;
                scanf("%d %d", &x, &y);
                printf("%d\n", Sum(x, y)%2);
            }
        }
        if (T)
            printf("\n");
    }
}
```
