---
title: UVA - 11212 Editing a Book（迭代加深搜索 IDA* + 模板）
date: 2017-02-21 20:53:02
categories: [ACM, 搜索]
tags:
---
# 题目链接
-------------------
https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=2153
# 题目大意
------------------
给定一个长度 1 ～ 9 的整数序列，每个整数 1 ～ 9 。

序列是无序的，你有两种操作，剪切和粘贴，两种操作均可以处理任意长度。

求至少经过多少次操作，可以使序列有序（递增）。

# 解题过程
--------------
本来感觉处理数组比较麻烦，每次都要开辟空间传递指针，然后赋值。

看了一个博客，可以用 string 类处理，然后感觉有现成的类，比手动辅助数组方便太多！！！

本题可以做一个迭代加深搜索的模板。

# 题目分析
------------------

#### 迭代加深搜索（IDA*）：
+ 适用范围：可用回溯法，但解答树没有明显上限的问题。
+ 从 0 开始枚举最大迭代深度，然后使用函数迭代求解，如果在当前最大迭代深度下有解，即是答案（处理最少步骤等问题）。
+ 切记，迭代加深搜索，解答树节点一般比较多，尽可能考虑剪枝。
+ 核心代码
```cpp
for (maxd = 0; ; maxd++)
{
	if (solve(0))
		break;
}

bool solve(int d)
{
	if (d == maxd)
		return judge();
	
	if (solve(d+1))
		return true;
		
	return false;
}
	
```
另外强调下 string 的 substr 方法。
substr(pos, len);
从 pos 开始， len 长度的子串，如果忽略 len ，则一直到结尾。

# AC代码
------------
```cpp
#include<bits/stdc++.h>
using namespace std;

int n, step, maxd;

string goal;

bool solve(int d, string now)
{
    if (d == maxd)
    {
        step = d;
        return now == goal;
    }

    int h = 0;
    for (int i = 0; i < n - 1; i++)
        if (now[i+1] - now[i] != 1)
            h++;

    if (h > 3 * (maxd - d))
        return false;

    for (int l = 0; l < n; l++)
    {
        for (int r = l; r < n; r++)
        {
            string slt = now.substr(l, r-l+1);
            string tail = now.substr(r+1);
            for (int k = 0; k < l; k++)
            {
                string il = now.substr(0, k);
                string ir = now.substr(k, l-k);
                if (solve(d+1, il+slt+ir+tail))
                    return true;
            }
        }
    }
    return false;
}

int main()
{
    int testCase = 0;
    while (~scanf("%d", &n) && n)
    {
        string data;
        for (int i = 0; i < n; i++)
        {
            int t;
            scanf("%d", &t);
            data += (char) t + '0';
        }

        goal = "";
        for (int i = '1'; i <= '0'+n; i++)
            goal += i;

        for (maxd = 0; maxd < n; maxd++)
            if (solve(0, data))
            {
                printf("Case %d: %d\n", ++testCase, maxd);
                break;
            }
    }
}

```
