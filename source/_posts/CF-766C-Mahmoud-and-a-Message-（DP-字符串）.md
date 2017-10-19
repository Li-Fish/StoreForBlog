---
title: CF 766C - Mahmoud and a Message （DP+字符串）
date: 2017-02-13 21:10:16
categories: [ACM, DP]
tags:
---
# 题目链接：
------------------
http://codeforces.com/problemset/problem/766/C
# 题目描述：
-------------
有一个只含小写字母的字符串，输入一个26个整数，用来限制每个字母所在字符串的最大长度，在保证符合限制的前提下。

+ 输出分割字符串的方案数。
+ 输出所有方案中，最少的分割次数。
+ 输出所有方案中，最大的子串长度。
# 结题过程：
------------
一开始打算放弃掉的，英文题，题目很长，看起来又比较麻烦。
然后学长来讲了一下，感觉毕竟讲都讲了，不做下挺可惜的，倒是个好题。

刚开始示例一直过不了，第二层循环字符串长度的时候出了问题，我用j表示到那一个字符了，后来发现用j代表分隔符的位置更好。

# 题目分析：
-----------
+ 首先确定状态，很明显字符串处理经常用长度做状态，无后效性。
+ 接下来是确定状态转移方程：
	+ 先说下第二层循环，这里j代表分隔符的所在位置，位于第j个字符的前方。
	用一个变量用来标记当前最小的长度限制，如果 **i - j + 1 > limit** 就跳出循环。
	最后虽然 **j = -1** 的时候退出循环，但是要给方案数加一，整个字符串可以不用分割。
	
	+ 方案次数：**dpNum[i] += dpNum[j-1] ,  0 < j <= i.**
	
	+ 最少子串数：**dpMinNum[i] = min(dpMinNum[j-1]+1, dpMinNum[i]).**
	+ 最长子串数，只需一直用 **j** 来更新最大长度 **ansLen** 就好了。
	
+ 最后注意一下初始化。

# AC代码：
---------
```cpp
#include<bits/stdc++.h>
using namespace std;

const long long MOD = 1e9+7;

int main()
{
    int n;
    scanf("%d", &n);
    char data[1123];
    int maxLen[30];
    scanf("%s", data);
    long long dpNum[1123], dpMinNum[1123], ansLen = 1;
    for (int i = 0; i < 26; i++)
        scanf("%d", maxLen+i);

    dpMinNum[0] = 1;
    dpNum[0] = 1;

    for (int i = 1; i < n; i++)
    {
        dpMinNum[i] = n;
        dpNum[i] = 0;
        int limit = n;
        int j;
        for (j = i; j >= 0; j--)
        {
            limit = min(maxLen[data[j]-'a'], limit);
            if (i-j+1 > limit)
                break;

            if (!i)
                continue;

            dpMinNum[i] =  min(dpMinNum[j-1]+1, dpMinNum[i]);
            ansLen = max(ansLen, (long long)i-j+1);
            dpNum[i] = (dpNum[i]%MOD + dpNum[j-1]%MOD)%MOD;
        }
        if (j == -1)
            dpNum[i]+=1;
    }
    printf("%lld\n%lld\n%lld\n", dpNum[n-1], ansLen, dpMinNum[n-1]);
}

```