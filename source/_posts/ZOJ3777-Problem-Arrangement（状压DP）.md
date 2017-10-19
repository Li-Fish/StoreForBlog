---
title: ZOJ3777 - Problem Arrangement（状压DP）
date: 2017-05-12 21:53:47
categories: [ACM, DP, 状压DP]
tags:
---
# 题目链接：
https://cn.vjudge.net/problem/ZOJ-3777

----------
# 题目大意：
 现在有一个N×N的矩阵，现在要求在这个矩阵里面取N个来自不同行不同列的数，使这个数大于给定的M。求总共有多少种取法。
(N < 12, M < 500)

-------------------------
# 解题过程：
 组队赛时候的题，当初是暴力DFS的，当作简化的八皇后问题，结果当然是超时。

 最近开始补题，正好在刷DP的题，于是顺手切了，算是一个状压DP的模板题。

-----------------------------

# 题目分析：

 首先看这个数据量，N和M的范围都非常小，显然可以拿来DP。一个题可以状态压缩，通常有几个量比较小，用来把状态压缩成二进制的数直接储存。

 以DP的思路来讲，这题的状态应该是二维的，一个维度是存的状态压缩后的集合，另一个是储存的当前累加的和，DP数组是存的当前解的个数。
 
 首先对于集合S，储存那些行已经有一个数被取了，这样可以保证一个无后效性，并且产生了许多重复子问题。如果前面已经取的点是集合S，并且前面的累加和是M。那么对于后面的数怎么取，有几个解，只需要计算一次即可。下一次如果一个状态仍是取的点的集合是S，并且累加和是M的话，那么直接用上一次计算的结果就可以了。

 这样总共的复杂度是***O（2^N*M）***，即状态总数。


---------------------

# 记忆化搜索：

```cpp
#include<bits/stdc++.h>
using namespace std;

#define LL long long

int dp[(1<<13)][600];
int n, m;
int data[112][112];

//计算阶乘
int fun(int n) {
    LL rst = 1;
    for (int i = 1; i <= n; i++) {
        rst *= i;
    }
    return rst;
}

int dfs(int dep, int num, int pre) {

	//递归边界
    if (dep >= n) {
        return num >= m;
    }

    int& ans = dp[pre][num];

    if (ans != -1)
        return dp[pre][num];

    ans = 0;

	//剪枝，如果当前已满足了，那么只需要计算下剩下的组合个数
    if (num >= m)
        return ans = fun(n-dep);

    for (int i = 0; i < n; i++) {
        if (pre&(1<<i))
            continue;
        int temp = pre|(1<<i);
        ans += dfs(dep+1, num+data[dep][i], temp);
    }
    return ans;
}



int main() {

    int T;
    scanf("%d", &T);
    while (T--) {
        memset(dp, -1, sizeof(dp));
        scanf("%d %d", &n, &m);

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                scanf("%d", &data[i][j]);
            }
        }

        int aa = fun(n);
        int bb = dfs(0, 0, 0);

        if (bb != 0) {
            int cc = __gcd(aa, bb);
            aa /= cc;
            bb /= cc;
            printf("%d/%d\n", aa, bb);
        }
        else {
            printf("No solution\n");
        }
    }
}
```

# 递推：

```cpp
#include<bits/stdc++.h>
using namespace std;

int dp[1<<13][666];
int data[112][112];

int fun(int n) {
    int rst = 1;
    for (int i = 2; i <= n; i++)
        rst *= i;
    return rst;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        int n, m;
        scanf("%d %d", &n, &m);
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                scanf("%d", &data[i][j]);
            }
        }
        memset(dp, 0, sizeof(dp));
        
        //代表对于前0行，使和为0的取法有1种
        dp[0][0] = 1;

		
		//遍历每一种取法
        for (int i = 0; i < (1<<n); i++) {
            int cnt = 0;
			
			//计算取到了第几行
            for (int j = 0; j < n; j++) {
                if (i&(1<<j))
                    cnt++;
            }
			
			//尝试取下一个数
            for (int j = 0; j < n; j++) {
                if (i&(1<<j))
                    continue;
				
				//遍历取到的和，如果和大于M，那么加到M那里
                for (int k = 0; k <= m; k++) {
					
					//这里的思想类似背包
                    if (k+data[cnt][j] >= m)
                        dp[i+(1<<j)][m] += dp[i][k];
                    else
                        dp[i+(1<<j)][k+data[cnt][j]] += dp[i][k];
                }
            }
        }
        int ans = dp[(1<<n)-1][m];
        if (ans == 0)
            printf("No solution\n");
        else {
            int aa = fun(n);
            int bb = __gcd(aa, ans);
            printf("%d/%d\n", aa/bb, ans/bb);
        }
    }
}
```