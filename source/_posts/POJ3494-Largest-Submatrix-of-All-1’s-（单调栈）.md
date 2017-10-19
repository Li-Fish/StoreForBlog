---
title: POJ3494 - Largest Submatrix of All 1’s （单调栈）
date: 2017-04-09 17:45:02
categories: [ACM, 数据结构, 栈]
tags:
---
# 题目链接：
 http://poj.org/problem?id=3494
 ---------------------
# 题目大意：
 给定一个矩阵，里面只包含 0 和 1，求这个矩阵里所有子矩阵中值最大是多少（即矩阵内部所有的 0，1 的和）。
 
---------------------------

# 解题过程：
 由于前几天刚做完一个单调栈的题，然后看到这个题就马上往那方面想了，但是还是不太熟练，对着模板才敲出来。

---------------------
# 题目分析：
+ 首先处理一下数据，开一个数组，数组 `data[i][j]` 代表着以第 i 行，第 j 列结尾的连续 1 的个数（自上到下的连续，例如`data[0][j], data[1][j]`都是 1 ，那么 `data[0][j] = 1, data[1][j] = 2`）。
+ 这样我枚举每一行，对于每一行，问题转化成了[一个典型的单调栈问题](http://poj.org/problem?id=2559)，求出所有行的最大值即是答案。


+ 这里说一下单调栈，在本题中，如果栈为空或者当前元素直接入栈，否则将栈顶元素出栈并计算栈顶元素的面积。
	+ 这里的面积是计算以栈顶元素为最大高度的矩阵面积，那么栈顶元素最左边可到达的是栈中上一个比他小的数（如果栈为空就是左边界了），最右可到达是即将要入栈的数（因为比他小），这里不直接那栈顶元素的下标和当前元素的下标计算是因为可能有大小相同的元素。
	
	+ 最后栈如果不为空的话要依次出栈并计算大小，计算大小的时候以栈中上一个比他小的数为最左可到达的地方（如果栈为空就是左边界了） ，最右可到达的是有边界，因为每一个栈中的数，都比右边所有的数要小。

# AC代码：
```cpp
#include<queue>
#include<cstdio>
#include<stack>
using namespace std;

const int MAX = 2000+100;

int data[MAX][MAX];

int main() {
    int m, n;
    while (~scanf("%d %d", &m, &n)) {
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                scanf("%d", &data[i][j]);
                if (data[i][j] != 0 && i != 0)
                    data[i][j] += data[i-1][j];
            }
        }

        int ans = 0;
        for (int i = 0; i < m; i++) {
            stack<int> ss;
            for (int j = 0; j < n; j++) {
                if (ss.empty() || data[i][j] > data[i][ss.top()]) {
                    ss.push(j);
                } else {
                    int start = ss.top();
                    ss.pop();
                    int width = ss.empty() ? j : j - ss.top()-1;
                    ans = max(ans, data[i][start]*width);
                    j--;
                }
            }
            while (!ss.empty()) {
                int start = ss.top();
                ss.pop();
                int width = ss.empty() ? n : n - ss.top() - 1;
                ans = max(ans, data[i][start]*width);
            }
        }
        printf("%d\n", ans);
    }
}

```