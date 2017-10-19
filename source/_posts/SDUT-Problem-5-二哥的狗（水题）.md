---
title: SDUT Problem_5 二哥的狗（水题）
date: 2016-12-26 22:45:59
categories: [ACM, 搜索]
tags:
---
# 题目链接：
[二哥的狗](http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Contest/contestproblem/cid/1970/pid/3420.html)
（在重现比赛里，可能随时消失，题目也找不到，这里把题目描述发一下好了）

第一学期已经接近尾声了，通往寒假的大门由八位Exam把守。
作为Exam大家族里的二哥，当然要比之前那些Exam们要凶恶的多。
二哥养了一群恶犬，如果你能战胜他们，二哥就放你过去。
当然不是让你赤手空搏，二哥也给你准备了同样数目的几只恶犬。
已知你的每只恶犬只能攻击二哥对应位置的恶犬，
当你的某只恶犬进行攻击时，会同时遭受敌方和其左右两只恶犬的攻击，如果某侧没有恶犬或者已被消灭，则该侧不会对你的恶犬造成伤害。
已知每只斗犬的攻击力，如果你的当前进行攻击的恶犬攻击力大于等于对方及左右两侧恶犬造成的总攻击力，就可以消灭掉敌方对应的那一只恶犬，否则不能消灭对方。


----------
# 题目大意：
就是输入来个等长的数组，然后把两个数组相同位置的数比较，假设输入a, b两个数组，如果b[i] >= a[i-1]+a[i]+a[i+1]，那么第一个数组的a[i]就会被攻击掉，不再提供战斗力了。


----------
# 解题过程：

 - 做这个题的曲线也是比较曲折的，花了将近半小时，如果考试的时候就GG了。
 - 刚开始一个思路时，从两部逐渐比较，比如定义head=0，tail定义尾部，然后先比较head，然后head++，再比较tail，tail--。
 - 这么想也是因为边界只有两条恶犬，赢得概率大点23333.
 - 然后WA了。
 - ![这里写图片描述](http://img.blog.csdn.net/20161226231423761?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 - 显然是不对的，随便可以出一组数据就GG了，比如
	 - 1 2 1
	 - 1 4 1
 - 如果这样先从两边往中间凑的话，两边都无法操纵就GG了，当初是为了O(n)的结果来的。
 - 然后就开始暴力两个for遍历就好了，显然是可以的，不过不知道我那里写错了，最后TLE，不过这样不太优雅~正好突然想起来一个思路，就是接下来题目分析上说的了，虽然TLE了一次，是因为已经攻击的狗又攻击了一次，处理一下就AC了。
 - 


----------
# 题目分析：

 - 首先一个循环把每个恶犬都遍历一遍。
 - 每条狗都尝试去进攻一下。
	 - 如果攻击成功的话就去让这条狗的左右两条狗去攻击下，因为一条狗攻击成功的话，那么它左右两条狗攻击成功所需要的攻击力就会刷新（减少）了。
	 - 如果不成功的话继续尝试下一条狗。
 - 最后结束上面的尝试后，再来一个for检查下二哥的狗是不是全部被咬死了就可以了。
 - 这样虽然最差的情况是O（n^2+n），但是感觉平均起来要好不少。

----------
# AC代码：

```cpp
#include<iostream>
#include<cstring>
using namespace std;

int n;
int data1[11234], data2[11234];

void attack(int i)
{
    if (data1[i] == 0)
        return;

    int temp = data1[i]+data1[i+1]+data1[i-1];
    if (data2[i] >= temp && temp)
    {
        data1[i] = 0;
        if (i-1 >= 1)
            attack(i-1);
        if (i+1 <= n)
            attack(i+1);
    }
}

int judge()
{
    for (int i = 1; i <= n; i++)
        attack(i);

    for (int i = 0; i <= n; i++)
        if (data1[i])
            return 0;

    return 1;
}

int main()
{
    while (cin >> n)
    {
        memset(data1,0,sizeof(data1));
        memset(data2,0,sizeof(data2));

        for (int i = 1; i <= n; i++)
            cin >> data1[i];
        for (int i = 1; i <= n; i++)
            cin >> data2[i];

        if (judge())
            cout << "Fighting!!!" << endl;
        else
            cout << "QAQ!!!" << endl;
    }
}
```