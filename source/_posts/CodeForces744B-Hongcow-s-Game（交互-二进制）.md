---
title: CodeForces744B - Hongcow's Game（交互+二进制）
date: 2017-08-18 21:25:01
categories: [ACM, 二进制]
tags:
---
# 题目链接：

http://codeforces.com/problemset/problem/744/B



--------------------
# 题目大意：

给出一个矩阵，矩阵主对角线元素均为 0，选择可以最多进行 20 次询问。

每次询问选择一些行，回答会输出每一行中这些列的最小的元素。

矩阵大小小于等于 1000 。



-------------------
# 解题过程：

想了半天没思路，到是有想到先询问一下奇数行和偶数行，然后没往后想...

也算是见识了一下新套路，这样利用位运算有点类似之前在知乎上看到的小白鼠测毒药的问题了233



--------------------
# 题目分析：

按二进制位将所有行分组，首先按最后一位分组，0为一组，1为一组，然后倒数第二位，以此类推，然后每次都查询一组里面所有的列。

这样这样考虑，对于第 i 行的第 j 列($j \ne i$)  ，既然不相等那么至少有一个二进制位是不同的，那么 i 和 j 至少有一次会被分到不同的组里面。这样对于所有行的每个元素都符合上面的描述，答案就可以求出来了。

对每次询问，如果这某一行的对角线所在的列也在分组中的话，就不更新答案。



----------------------
# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

const int INF = 0x3f3f3f3f;
const int MAX = 1123;

int ans[MAX];

int main() {
    int n;
    cin >> n;
    memset(ans, INF, sizeof(ans));

    //数据不会超过1000
    for (int i = 0; i < 10; i++) {
        vector<int> data1, data2;
        for (int j = 1; j <= n; j++) {
            //按二进制位为0或1分组
            if (j & (1 << i)) data1.push_back(j);
            else data2.push_back(j);
        }

        //询问第一组
        if (data1.size()) {
            cout << data1.size() << endl;
            for (int j = 0; j < data1.size(); j++) {
                cout << data1[j] << " ";
            }
            for (int j = 1; j <= n; j++) {
                int t;
                cin >> t;
                //对对角线不在分组中的列更新答案
                if ((j & (1 << i)) == 0) {
                    ans[j] = min(ans[j], t);
                }
            }
            cout << endl;
            fflush(stdout);
        }

        //询问第二组
        if (data2.size()) {
            cout << data2.size() << endl;
            for (int j = 0; j < data2.size(); j++) {
                cout << data2[j] << " ";
            }
            for (int j = 1; j <= n; j++) {
                int t;
                cin >> t;
                if (j & (1 << i)) ans[j] = min(ans[j], t);
            }
            cout << endl;
            fflush(stdout);
        }
    }

    fflush(stdout);
    cout << -1 << endl;
    for (int i = 1; i <= n; i++) cout << ans[i] << " ";
}
```