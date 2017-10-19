---
title: UVA - 1606 Amphiphilic Carbon Molecules（极角排序+扫描法+计算几何）
date: 2017-04-03 16:27:26
categories: [ACM, 计算几何]
tags:
---
# 题目链接：
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=4481

---------------------------------------
# 题目大意：
 平面上有 n 个点， 每个点为白点或黑点。现在需要放置一条隔板，使得隔板一侧的黑色点数加上另一侧白色点数的和最大。

------------------------------------
# 解题过程：
 这是人生第一道 AC 的计算几何题！
 由于之前一直没接触过这种题，紫书上的分析也看不懂，一言不合就让我极角排序、叉乘，根本没学过啊QAQ。
 然后翻了很多博客，重学了一下极坐标，可算是弄明白了。

------------------------------------------
# 题目分析：
 首先这题最简单的想法是枚举所有可能的挡板，这样需要枚举两个点，然后统计两端的点。枚举是 n×n， 统计也是 n，总共时间复杂度是 n^3。显然是超时的。

 然后扫描法的思想是，按照一定顺序枚举，这样可以动态的维护一个量，省去了扫描时的 n。

 所以这题正解是枚举每一个点当作基点，然后计算每个点相对于基点的极角，进行极角排序。然后按极角的大小进行旋转时的枚举。每旋转一次可以根据之前统计的点的个数进行动态的更新（这就是“维护”）。

 这题有个可以优化的地方，如果一个点是黑点的话，可以把他映射到关于基点对称的地方去，视为白点。这样扫描的时候只需要统计白点就够了。然后扫描的时候只从扫描 180 度即可。

 判断是否超过 180 度用叉乘，**a**×**b** = `|a| * |b| * sin<a,b>`，两个点叉乘用坐标计算是，`x1*y2 - y1*x2` 因为向量的模一定是正的，所以只要判断这个叉乘正负，就可以判断 sin 的正负了，即是否超过 180 度。

----------------------------------

# AC代码：
```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 1005;

struct Node {
    int x, y;
    int color;
    double rad;
}raw_data[MAX], data[MAX];

bool operator < (const struct Node &a, const struct Node &b) {
    return a.rad < b.rad;
}

int n, ans;

//用叉乘判断角度是否大于180度，即sin值。
bool Turn(Node &a, Node &b) {
    return a.x * b.y >= a.y * b.x;
}

void solve() {
    if (n <= 3) {
        ans = n;
        return;
    }
    ans = 0;

    //枚举每一个点当多基点
    for (int i = 0; i < n; i++) {
        int k = 0;
        for (int j = 0; j < n; j++) {
            if (i == j)
                continue;

            //计算每个点关于基点的相对坐标。
            data[k].x = raw_data[j].x - raw_data[i].x;
            data[k].y = raw_data[j].y - raw_data[i].y;

            //如果是黑点的话，那么映射到关于基点对称的地方，这样扫描的时候只记录白点就够了。
            if (raw_data[j].color) {
                data[k].x *= -1;
                data[k].y *= -1;
            }
            //求极角
            data[k].rad = atan2(data[k].y, data[k].x);
            k++;
        }

        //极角排序
        sort(data, data + k);

        //L为与基点相连的点，R为扫描线。
        int L = 0, R = 0, cnt = 1;

        //枚举每一个点与基点相连当作挡板。
        while (L < k) {
            if (L == R) {
                R = (R + 1) % k;
                cnt++;
            }

            //扫描的时候只扫描一侧，保证扫描线与挡板的夹角不超过180度。
            while (L != R && Turn(data[L], data[R])) {
                cnt++;
                R = (R + 1) % k;
            }
            ans = max(ans, cnt);
            //挡板旋转。
            L++;
            //由于挡板旋转，与基点相连的点不在挡板上了，减去1。
            cnt--;
        }
    }
}

int main() {
    while (cin >> n && n) {
        for (int i = 0; i < n; i++) {
            cin >> raw_data[i].x >> raw_data[i].y >> raw_data[i].color;
        }
        solve();
        cout << ans << endl;
    }
}

```