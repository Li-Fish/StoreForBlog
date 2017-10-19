---
title: Uva 806 - Spatial Structures（四分树+模板）
date: 2017-02-09 14:58:59
categories: [ACM, 数据结构]
tags:
---
# 题目链接：
-----
https://vjudge.net/problem/24840/origin

# 题目描述：
-------------
黑白图像有好几种表示方式，可以用点阵或者路径，路径需要把图像转化成四叉树。
题目要求就是实现两种表示方式的转换。

# 解题过程：
-----
题目不难，注意细节就好，直接的judge函数错了好几次，以后写的严谨点，不拿两个相邻的方块比较了，立一个flag再和flag比较。
没注意输出格式，pe了好几次，12个数字为一行。
 
# 题目分析：
----
+ 转化为路径，首先以递归的方式建树，每次搜索范围缩小为原来的1/4。
找到黑块的时候，把搜索时记录下来的路径转化成十进制，储存。
最后排序输出，12个为一行。
+ 转化为图像，首先把十进制转化为路径，然后递归式的把图像画出来就好了。

# AC代码：
-------
```cpp
#include <cstdio>
#include <cstring>
#include <map>
#include <cmath>
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;

struct node
{
    int status;
    node *one, *two, *three, *four;
    node():status(-1),one(NULL),two(NULL),three(NULL),four(NULL){}
};

vector<int> ans;
int num;
string drawPath;
char data[112][112];

bool judge(int l, int r, int u, int d)
{
    int flag = data[u][l];
    for (int i = u; i < d; i++)
        for (int j = l; j < r; j++)
            if (data[i][j] != flag)
                return false;
    return true;
}

void trns(int num)
{
    drawPath = "";
    if (num == 0)
    {
        drawPath += '0';
        return;
    }

    while (num)
    {
        string temp = "";
        temp += (char) (num%5 + '0');
        drawPath = temp + drawPath;
        num /= 5;
    }

//    cout << drawPath << endl;
}

void add(string path)
{
    int len = path.size(), num = 0;
    for (int i = 0; i < len; i++)
        num += (path[i] - '0') * pow(5, i);

    ans.push_back(num);
}

void solve(int l, int r, int u, int d, string path)
{

    if (judge(l, r, u, d))
    {
        if (data[u][l] == '1')
        {
            add(path);
            num++;
        }
        return;
    }

    solve(l, (l+r)/2, u, (u+d)/2, path+'1');
    solve((l+r)/2, r, u, (u+d)/2, path+'2');
    solve(l, (l+r)/2, (u+d)/2, d, path+'3');
    solve((l+r)/2, r, (u+d)/2, d, path+'4');
}

void draw(int l, int r, int u, int d, int p)
{
    if (p == -1 || drawPath[p] == '0')
    {
        for (int i = u; i < d; i++)
            for (int j = l; j < r; j++)
                data[i][j] = '*';
        return;
    }
//    printf("%c\n", drawPath[p]);
    if (drawPath[p] == '1')
        draw(l, (l+r)/2, u, (u+d)/2, p-1);
    if (drawPath[p] == '2')
        draw((l+r)/2, r, u, (u+d)/2, p-1);
    if (drawPath[p] == '3')
        draw(l, (l+r)/2, (u+d)/2, d, p-1);
    if (drawPath[p] == '4')
        draw((l+r)/2, r, (u+d)/2, d, p-1);
}

int main()
{
//    freopen("in", "r", stdin);
//    freopen("out", "w", stdout);
    int n, testCase = 0;
    while (~scanf("%d", &n) && n)
    {
        if (testCase)
            putchar('\n');
        printf("Image %d\n", ++testCase);
        num = 0;
        if (n > 0)
        {
            ans.clear();
            for (int i = 0; i < n; i++)
                scanf("%s", data+i);
            solve(0, n, 0, n, "");
            sort(ans.begin(), ans.end());
            for (int i = 0; i < ans.size(); i++)
            {
                printf("%d", ans[i]);
                if ((i+1) % 12 == 0)
                    printf("\n");
                else if (i != ans.size()-1)
                    putchar(' ');
            }

            if (num && ans.size()%12 != 0)
                putchar('\n');
            printf("Total number of black nodes = %d\n", num);
        }
        else
        {
            memset(data, '.', sizeof(data));
            n *= -1;
            int t;
            while (cin >> t && t != -1)
            {
                trns(t);
                draw(0, n, 0, n, drawPath.size()-1);
            }


            for (int i = 0; i < n; i++)
            {
                for (int j = 0; j < n; j++)
                    putchar(data[i][j]);
                putchar('\n');
            }
        }
    }
}
```