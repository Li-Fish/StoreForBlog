---
title: HDOJ1176. 免费馅饼.（DP）
date: 2016-12-05 22:05:33
categories: [ACM, DP]
tags:
---
# 题目链接：
[免费馅饼](http://acm.hdu.edu.cn/showproblem.php?pid=1176)


----------
# 题目大意：
![这里写图片描述](http://acm.sdut.edu.cn/image/1366.jpg)
初始位置为5，输入时间和位置，从1秒开始，每次可以移动一个位置，一秒时可接住4,5,6处馅饼，求最大接住馅饼数。


----------
# 解题过程：

 - 刚开始就找到了状态转移方程：
 - j代表位置，i代表时间
  ***dp[j][i] = max(dp[j-1][i], dp[j-1][i-1], dp[j-1][i+1]) + data[j][i]; （j点位置曾经达到过）***
  
 - 数塔是没想到，看到题解后才发现原来这可以归一类问题
 - 不过没想到状态转移方程出来后题目还是WA，莫名其妙，反复改后变成TLE了，介于这题cin都会TLE估计是卡了常数……
 - 于是发现别人数塔都是从后往前数的于是改了下状态转移方程A了，这样也不需要开data数组了。
 - 于是找了下之前数塔的题，发现直接都是从前往后数的，或者多开了一个数组……算是学到了。


----------
# 题目分析：

 - 就是一个数塔
 -  j代表位置，i代表时间
 - 状态转移方程：
  ***dp[j][i] = max(dp[j][i+1], dp[j-1][i+1], dp[j+1][i+1]) + dp[j][i]***


----------
# AC代码：

```cpp
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;

int dp[20][1123456];

int MaxOf3(int a, int b, int c){
    int max = (a > b) ? a : b;
    return (max > c) ? max : c;
}

int MaxOf2(int a, int b){
    return (a > b) ? a : b;
}

int main()
{
    int n;
    while ((scanf("%d", &n) != EOF) && n)
    {
        int maxt = 0, position, time;

        memset(dp, 0, sizeof(dp));

        for (int i = 0; i < n; i++)
        {
            scanf("%d %d", &position, &time);
            dp[position+1][time]++;
            if (maxt < time)
                maxt = time;
        }

        for (int i = maxt; i >= 0; i--)
        {
            for (int j = 1; j < 12; j++)
            {
                dp[j][i] = MaxOf3(dp[j][i+1], dp[j-1][i+1], dp[j+1][i+1]) + dp[j][i];
            }
        }

        cout << dp[6][0] << endl;
    }
}
```

# 从前往后TLE的错误代码：

```
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;

int dp[20][1123456];
int data[20][1123456];

int MaxOf3(int a, int b, int c){
    int max = (a > b) ? a : b;
    return (max > c) ? max : c;
}

int MaxOf2(int a, int b){
    return (a > b) ? a : b;
}

int main()
{
    int n;
    while ((scanf("%d", &n) != EOF) && n)
    {
        int maxt = 0, position, time;

        memset(dp, -1, sizeof(dp));
        memset(data, 0, sizeof(data));
        dp[6][0] = 0;

        for (int i = 0; i < n; i++)
        {
            scanf("%d %d", &position, &time);
            data[position+1][time]++;
            if (maxt < time)
                maxt = time;
        }


        for (int i = 1; i <= maxt; i++)
        {
            for (int j = 1; j < 12; j++)
            {
                dp[j][i] = MaxOf3(dp[j][i-1], dp[j-1][i-1], dp[j+1][i-1]);
                if (dp[j][i] != -1)
                    dp[j][i] += data[j][i];
            }
        }

        int ans = -1;
        for (int i = 0; i < 12; i++)
        {
            if (ans < dp[i][maxt])
                ans = dp[i][maxt];
        }
        cout << ans << endl;
    }
}
```