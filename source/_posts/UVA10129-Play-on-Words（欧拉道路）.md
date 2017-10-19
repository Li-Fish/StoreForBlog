---
title: UVA10129-Play on Words（欧拉道路）
date: 2016-12-15 16:32:06
categories: [ACM, 图论]
tags:
---
# 题目链接：


[UVA10129-Play on Words](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1070)

----------
# 题目大意：
给出一些单词，如果一个单词的首部字母和另一个单词的尾部字母相同，则可以首尾连接，每个单词只能使用一次，判断这些字母是否可以全部连接成一个串。


----------
# 解题过程：

 - 刚开始没弄清有向图和无向图欧拉路的定义，以为只判断边就可以了，于是简单写了一个只判读边的。
 - 显然是错误的，自己试了几组数据就不对。
 [我这样想可能是被这个题带偏了节奏（误](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Index/problemdetail/pid/3683.html)
 
 - 然后仔细看了下书上的定义，还要保证图必须是连同的才可以，于是写了下，一发AC。
 - 


----------
# 题目分析：

 - **首先明确欧拉路的充要条件：**
 -  对于无向图，最多只有两个点的度为1，则一定存在欧拉回路。
 - 对于有向图，最多只能有两个点的入度不等于出度，而且一个是入度恰好比出度大1，另一个是出度比入度大1。
 - 以上两个图都要保证**底图**（忽略边的方向后得到的图）是连通的。
 - 如果一个图不存在**奇点**（度数为奇数的点）则构成欧拉回路，否则为欧拉路。
 


----------

 - 题目是有向图，把每个字母当做点，用两个数组统计每个点入度和出度，并判断。
 - 用dfs的方法把图遍历一遍，看是否每个点都到达过，是的话则是连通的图。
 - 两个判断条件与运算即是结果。
 - 


----------
# AC代码：

```cpp
#include<cstdio>
#include<iostream>
#include<cstring>
#include<algorithm>
using namespace std;

int head[26], tail[26], data[26][26], mark[26];

bool dfs(int k)
{
    mark[k] = 0;
    for (int i = 0; i < 26; i++)
    {
        if (data[k][i])
        {
            data[k][i] = 0;
            data[i][k] = 0;
            dfs(i);
        }
    }
}

bool judge()
{
    int k;
    for (int i = 0; i < 26; i++)
    {
        if (mark[i] != 0)
        {
            k = i;
            break;
        }
    }
    dfs(k);
    for (int i = 0; i < 26; i++)
        if (mark[i])
            return false;
    return true;
}

int main()
{
    int T;
    cin >> T;
    while (T--)
    {
        memset(head, 0, sizeof(head));
        memset(tail, 0, sizeof(tail));
        memset(data, 0, sizeof(data));
        memset(mark, 0, sizeof(mark));
        int n, flag = 1;
        cin >> n;
        string str;
        while (n--)
        {
            cin >> str;
            head[str[0]-'a']++;
            tail[str[str.size()-1]-'a']++;

            data[str[0]-'a'][str[str.size()-1]-'a'] = 1;
            data[str[str.size()-1]-'a'][str[0]-'a'] = 1;

            mark[str[0]-'a'] = 1;
            mark[str[str.size()-1]-'a'] = 1;
        }
        int a = 0, b = 0;
        for (int i = 0; i < 26; i++)
        {
            if (head[i]-tail[i] == 1)
                a++;
            if (tail[i]-head[i] == 1)
                b++;
            if (abs(head[i]-tail[i]) > 1)
                flag = 0;
        }
        if (flag && a == b && a <= 1 && judge())
            cout << "Ordering is possible." << endl;
        else
            cout << "The door cannot be opened." << endl;
    }
}

```


----------
# 扩展应用：
按书上的说法欧拉回路的来源好像就是为了解决一笔画问题，于是写了一个有意思的程序，给定一个含有欧拉路的图，求如何打印欧拉路。


----------
# 功能用法：
首先打开一个游戏
![这里写图片描述](http://img.blog.csdn.net/20161215170447246?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后分析出有几条边和点。
![这里写图片描述](http://img.blog.csdn.net/20161215170542887?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

输入数据，即可得出结果。
![这里写图片描述](http://img.blog.csdn.net/20161215170612992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

----------
# 源代码：
```
#include <iostream>
#include <cstring>
#include <stack>
using namespace std;

const int MAX = 1123;

int m, n;
int data[MAX][MAX];
stack<int> ans;

void dfs(int pos)
{
    for (int i = 1; i <= n; i++)
    {
        if (data[pos][i])
        {
            data[pos][i] = 0;
            data[i][pos] = 0;
            dfs(i);
        }
    }
    ans.push(pos);
}

void put()
{
    while (!ans.empty())
    {
        cout << "->" << ans.top();
        ans.pop();
    }
    cout << endl;
}

int main()
{
    while (cin >> n >> m)
    {
        memset(data, 0, sizeof(data));
        int a, b;
        for (int i = 0; i < m; i++)
        {
            cin >> a >> b;
            data[a][b] = 1;
            data[b][a] = 1;
        }
        dfs(1);
        put();
    }
}
```