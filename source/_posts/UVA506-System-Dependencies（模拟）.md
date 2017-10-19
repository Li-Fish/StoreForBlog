---
title: UVA506 System Dependencies（模拟）
date: 2017-02-07 19:00:11
categories: [ACM, 模拟]
tags:
---
# 题目链接：
https://vjudge.net/problem/31990/origin


----------
# 题目大意：
模拟安装软件，分显式好隐式安装两种情况。安装一个软件的时候，指明安装的就是显式安装。
比如：

> INSTALL    A 

这里A是显示安装，如果A必须要B和C软件的支持的话，顺便安装上B和C，这里B和C就是隐式安装上的。

然后是删除软件，删除的时候以递归的形式删除，先删除指明安装的软件，然后删除这个软件的支持软件，但是如果支持软件是显式安装上的话，就不删除了。

> INSTALL   B
> INSTALL   A
> REMOVE   A 


这里的B就不会被删除，安装A时同时安装的C会被删除掉。此外如果A的支持软件是另一个软件的支持软件的话，也不会被删除掉。


----------
# 解题过程：
题目本身不难，有是一个复杂的模拟题，书上给了些提示和代码，对比着，写出来不难。
需要映射下软件的名字，这里用到了之前 UVA-12096 里面的映射方法。

----------
# 题目分析：
先把字符串名称用MAP映射成整数，容易处理些。
安装和删除的函数，注意递归处理。
把depend的输入分离的时候使用stringstream处理。

题目不难，不过用到了好多有用的方法，这里Mark下。

----------
# AC代码：

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAX = 112345;

map<string, int> nameList;
vector<string> getName, nameTable;
string str;
vector<int> depend1[MAX], depend2[MAX];
int status[MAX];
string blank = "   ";

int ID(string name)
{
    if (nameList.count(name))
        return nameList[name];

    getName.push_back(name);
    return nameList[name] = getName.size()-1;
}

void depend()
{
    stringstream ss;
    string item_name;
    int item1, item2;
    ss << str;
    ss >> item_name;

    ss >> item_name;
    item1 = ID(item_name);

    while (ss >> item_name)
    {
        item2 = ID(item_name);
        depend1[item1].push_back(item2);
        depend2[item2].push_back(item1);
    }
}

void install(int item, bool topLevel)
{
    if (!status[item])
    {
        for (int i = 0; i < depend1[item].size(); i++)
            install(depend1[item][i], false);

        string item_name = getName[item];
        cout << blank << "Installing " << item_name << endl;
        nameTable.push_back(item_name);
        status[item] = topLevel? 1:2;
    }
    else if (topLevel)
        cout << blank << getName[item] << " is already installed." << endl;
}

void list_()
{
    for (int i = 0; i < nameTable.size(); i++)
        cout << blank << nameTable[i] << endl;
}

bool needed(int item)
{
    for (int i = 0; i < depend2[item].size(); i++)
        if (status[depend2[item][i]])
            return true;
    return false;
}

void remove_(int item, bool topLevel)
{
    string item_name = getName[item];
    if (status[item] == 0 && topLevel)
    {
        cout << blank << item_name << " is not installed." << endl;
        return;
    }

    if (!needed(item) && (status[item] == 2 || topLevel))
    {
        status[item] = 0;
        for (int i = 0; i < nameTable.size(); i++)
        {
            if (nameTable[i] == item_name)
            {
                nameTable.erase(nameTable.begin()+i);
                break;
            }
        }
        cout << blank << "Removing " << item_name << endl;

        for (int i = 0; i < depend1[item].size(); i++)
            remove_(depend1[item][i], false);
    }
    else if (topLevel)
        cout << blank << item_name << " is still needed." << endl;
}

int main()
{
//    freopen("in","r",stdin);
    while(getline(cin, str))
    {
        cout << str << endl;
        if (str[0] == 'D')
        {
            depend();
        }

        if (str[0] == 'I')
        {
            stringstream ss;
            ss << str;
            string item_name;
            ss >> item_name;
            ss >> item_name;

            int item = ID(item_name);

            install(item, true);
        }

        if (str[0] == 'E')
        {
            break;
        }

        if (str[0] == 'R')
        {
            stringstream ss;
            ss << str;
            string item_name;
            ss >> item_name;
            ss >> item_name;

            int item = ID(item_name);

            remove_(item, true);
        }

        if (str[0] == 'L')
        {
            list_();
        }
    }
}

```