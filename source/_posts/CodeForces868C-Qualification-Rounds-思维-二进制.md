---
title: CodeForces868C - Qualification Rounds(思维+二进制)
date: 2017-10-09 16:41:24
categories:
tags:
---
# 题目链接：

[http://codeforces.com/contest/868/problem/C](http://codeforces.com/contest/868/problem/C)

# 题目大意：

现在有 n 个长度为 k 二进制串，现在要从中选出 m 个二进制串，使得每一位的 1 的数量不能大于 0 的数量。

$1 \le n \le 10^5, 1 \le k \le 4$



# 解题过程：

比赛的时候一直没思路，于是乱写了一个假算法，对每个串分配权重并贪心，不过居然能过，不过还是被 Hack 掉了。

# 题目分析：

首先，这里 K 非常小，所以最多也就是有 16 种不同的串，考虑用桶来统计。

然后假设有一个解，那么可以[证明](http://www.cnblogs.com/yyf0309/p/7632780.html)，至多只需要选两个串就可以满足题目要求（如果存在 0000 的串，那么只需这一个串就可以满足题目要求了）。

那么这时候我们暴力枚举这如果串就可以了，复杂度$O(16\times 16)$ 




# AC代码：
```cpp
#include <bits/stdc++.h>

using namespace std;

//统计每个串的出现次数
int data[1123];

int main() {
    int n, k;
    cin >> n >> k;
    while (n--) {
        int temp = 0, cnt = 1;
        for (int i = 1; i <= k; i++) {
            int t;
            cin >> t;
            temp += t * cnt;
            cnt *= 2;
        }
        data[temp]++;
    }
    //如果存在 0000 这样的串，那么只选这一个串就可以了
    if (data[0]) {
        printf("YES\n");
        return 0;
    } else {
        //枚举如果串
        for (int i = 0; i < (1 << k); i++) {
            if (!data[i]) continue;
            data[i]--;
            for (int j = 0; j < (1 << k); j++) {
                //这里要保证两个串的每一位至少有一个是 0，那么他们 and 操作结果是 0
                if ((i & j) == 0 && data[j]) {
                    printf("YES\n");
                    return 0;
                }
            }
            data[i]++;
        }
    }
    printf("NO\n");
}
```