---
title: POJ1015 - Jury Compromise（DP+计算顺序）
date: 2017-07-02 10:39:08
categories: [ACM, DP]
tags:
---
# 题目链接：
http://poj.org/problem?id=1015

-----------------------------------
# 题目大意：
现在两个长度相等的序列，$D$和$P$，现在要构造一个新的序列$A = \lbrace a_1, a_2, a_3 \cdots a_k\rbrace$使得$\big|\sum_{i=1}^kD[a_i]-\sum_{i=1}^kP[a_i]\big|$尽量的小，如果有多解，要求$\sum_{i=1}^k(D[a_i]+P[a_i])$尽量的大。

-------------------------------------
# 解题过程：
历经两天才A掉的这个题，刚开始定义的三维的状态，果断超时了，后来建去以为，以当前差值的绝对值为状态，但是这样好像会失解，最后看了下博客，发现不用取绝对值可以了。

但是这题要求输出路径，于是纠结好久如何输出路径，要保证一个元素不难重复使用的话，只能从后往前递推，如何这样之前的路径就可能被之后更新掉，原因应该是，以这样DP，不符合最优子结构，全局最优解不是局部最优解，然后局部的解被更新掉之后，输出路径的时候就错了。

最后又去看了下博客，发现别人都是从前往后递推，并且转移的时候检查下当前元素有没有被用过。从前往后推的话，就能保证当前状态向后找的路径是确定不变的了。


----------------------------------------
# 题目分析：
定义状态$dp[i][j]$为$\sum_{i=1}^jD[a_i]-\sum_{i=1}^jP[a_i]$的结果为$i$时最大的$\sum_{i=1}^kD[a_i]+P[a_i]$。

那么状态之间转移为：
设当前已选的元素的集合为$D$

$$dp[i+D[k]-P[k]][j+1] = max(dp[i][j]+D[k]+P[k]) \wedge 1 \le k \le n\wedge k\notin D$$


关键在于如何判断$k\notin D$，这里只需要去递归访问路径，查看$k$是否在当前状态的路径中。

做这道题的时候，真的意识到写递推$DP$时计算顺序是多么重要，要考虑循环的嵌套顺序，是循环变量从前往后还是从后向前，一个不同，含义就改变了许多。

---------------------------------------
# AC代码
```cpp
#include <cstring>
#include <cstdio>
#include <vector>
#include <cmath>
#include <algorithm>
using namespace std;

const int INF = 0x3f3f3f3f;

int reserve1[3123][30], reserve2[3123][30];
int (*dp)[30] = reserve1+1512;
int (*pre)[30] = reserve2+1512;
int D[212], P[212];

//判断k是否已经被选择
bool is_select(int i, int j, int k) {
    while (~pre[i][j]) {
        int t = pre[i][j];
        if (t == k)
            return true;
        t = D[t]-P[t];
        i -= t;
        j--;
    }
    return false;
}

int main() {
    int n, m, cases = 0;
    while (~scanf("%d %d", &n, &m) && (n+m)) {
        for (int i = 0; i < n; i++) {
            scanf("%d %d", D+i, P+i);
        }
        memset(reserve1, 0x80, sizeof(reserve1));
        memset(reserve2, -1, sizeof(reserve2));
        //初始化边界状态
        dp[0][0] = 0;

        //记录答案
        int ans_diff, ans_sum;
        vector<int> ans_path;
        ans_diff = INF, ans_sum = -INF;

        for (int i = -1000; i <= 1000; i++) {
            for (int j = 0; j < m; j++) {
                //如果当前为负数，表示当前状态不可到达
                if (dp[i][j] < 0) continue;
                for (int k = 0; k < n; k++) {
                    int t1 = D[k]+P[k];
                    int t2 = D[k]-P[k];
                    if (!is_select(i, j, k) && (dp[i][j] + t1 > dp[i+t2][j+1])) {
                        dp[i+t2][j+1] = dp[i][j] + t1;
                        pre[i+t2][j+1] = k;
                        //当j等于m时更新答案
                        if (j+1 == m && (abs(i+t2) < abs(ans_diff) || abs(i+t2) == abs(ans_diff) && dp[i+t2][j+1] > ans_sum)) {
                            ans_diff = i+t2;
                            ans_sum = dp[i+t2][j+1];
                        }
                    }
                }
            }
        }

        int sum1, sum2;
        sum1 = sum2 = 0;
        int pos = m;
        //递归的去寻找路径
        while (~pre[ans_diff][pos]) {
            int t = pre[ans_diff][pos];
            ans_path.push_back(t);
            sum1 += D[t];
            sum2 += P[t];
            t = D[t]-P[t];
            ans_diff -= t;
            pos--;
        }

        sort(ans_path.begin(), ans_path.end());
        printf("Jury #%d\n", ++cases);
        printf("Best jury has value %d for prosecution and value %d for defence:\n", sum1, sum2);
        for (int i = 0; i < ans_path.size(); i++)
            printf(" %d", ans_path[i]+1);
        printf("\n\n");
    }
}
```