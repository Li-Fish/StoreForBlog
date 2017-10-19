---
title: DAG上的动态规划
date: 2017-04-26 10:49:13
categories: [ACM, DP]
tags:
---
# 不定终点的最长路：
--------------
## 矩阵嵌套问题：
--------------------------
### 题目链接：
http://acm.nyist.net/JudgeOnline/problem.php?pid=16

------------------------
### 题目大意：
 如果一个矩形的长和宽都大于另一个矩形，那么说这个矩形可以嵌套另一个矩形，现在有很多个矩形，输入矩形的长宽，求出他们的最大嵌套数。注意矩形的长宽是可以互换的。

---------------------
### 题目分析：
 这是一道经典的DAG上的动态规划问题，需要把问题转化成DAG上的最长路问题，把每个矩形看成点，如果两个矩形可以嵌套，那么他们建边，这样问题就是求最长路了。

 状态的定义是对于第 i 个节点，由他出发最长的距离，根据边转移，如果一个矩形可以装下另一个矩形，那么这个矩形出发的最长距离就是他可以装下的那个矩形的最长距离加一。

 递归边界是隐含的，如果一个矩形无法嵌套嵌套的矩阵，那么由他出发的最长距离是 1 ，即只有他本身。

---------------------------
### AC代码：
```cpp

#include<bits/stdc++.h>
using namespace std;

int edge[1123][1123];
int d[1123], n;

int dp(int i) {
    int& ans =  d[i];
    if (ans > 0)
        return ans;
    ans = 1;
    for (int j = 0; j < n; j++) {
        if (edge[i][j])
            ans = max(ans, dp(j)+1);//状态转移
    }
    return ans;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        int data[1123][2];
        for (int i = 0 ; i < n; i++) {
            scanf("%d %d", &data[i][0], &data[i][1]);
            if (data[i][0] > data[i][1])
                swap(data[i][0], data[i][1]);
        }

        memset(edge, 0, sizeof(edge));
        memset(d, 0, sizeof(d));

        for (int i = 0; i < n; i ++) {
            for (int j = 0; j < n; j++) {
                if (i == j)
                    continue;
                if (data[i][0] > data[j][0] && data[i][1] > data[j][1])
                    edge[i][j] = 1;//如果可以嵌套那么建边
            }
        }

        int ans = 0;
        for (int i = 0; i < n; i++) {
            ans = max(ans, dp(i));//最终答案是所有节点出发最长的那个
        }

        printf("%d\n", ans);
    }
}

```

## The Tower of Babylon

---------------------------------
### 题目链接：
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=378

------------------------
### 题目大意：
 现在有很多立方体，每个立方体输入他的三个棱长即可确定，现在规定如果一个立方体的面的长和宽都大于另一个立方体的长和宽，那么这两个立方体是可以堆叠起来的。现在输入很多立方体，求堆叠出的最高高度。注意两个立方体三个棱长都要考虑，两个棱长确定是否可堆叠，一个决定堆起来的高度。立方体可以重复使用。

---------------------------
### 题目分析：
 和上一题近似，不过这里我可以把每个立方体拆成三个，分别对于三个不同的面和高，然后根据面的长宽大小，对他们建边。然后就是求DAG上的最长路问题，不过这题的不同是，每条边有权了，并且不能直接拿长宽的大小当数组的下标。

-----------------------------
### AC代码：

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Node {
    int a, b;
    int h;
    Node(int a, int b, int h) {
        if (a > b)
            swap(a, b);
        this->a = a;
        this->b = b;
        this->h = h;
    }
};

bool operator < (const struct Node& a, const struct Node& b) {
    if (a.a < b.a && a.b < b.b)
        return true;
    return false;
}

vector<Node> data;
int edge[1123][1123];
int dp[1123];

int dfs(int u) {
    int& rst = dp[u];
    if (rst >= 0)
        return rst;
    rst = 0;
    for (int i = 0; i < data.size(); i++) {
        if (edge[u][i])
            rst = max(rst, dfs(i));
    }
    //最后要加上当前的立方体的高度
    return rst += data[u].h;
}

int main() {
    int n, cases = 0;
    while (~scanf("%d", &n) && n) {
        data.clear();
        for (int i = 0; i < n; i++) {
            int a, b, c;
            scanf("%d %d %d", &a, &b, &c);
            //把立方体根据每个面拆成三个
            data.push_back(Node(a, b, c));
            data.push_back(Node(a, c, b));
            data.push_back(Node(b, c, a));
        }
        memset(edge, 0, sizeof(edge));
        for (int i = 0; i < data.size(); i++) {
            for (int j = 0; j < data.size(); j++) {
                //如果可以堆叠，那么建边
                if (data[i] < data[j]) {
                    edge[i][j] = 1;
                }
            }
        }

        memset(dp, -1, sizeof(dp));
        int ans = 0;
        for (int i = 0; i < data.size(); i++) {
            ans = max(dfs(i), ans);
        }
        printf("Case %d: maximum height = %d\n", ++cases, ans);
    }
}
```