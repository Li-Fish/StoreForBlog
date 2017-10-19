---
title: 自动走迷宫（DFS）
date: 2016-12-05 23:12:23
categories: [ACM, 搜索]
tags:
---
# 功能：
 - 输入一张地图，找出一条可到达终点的路径，1代表不可走，0代表可走，2代表终点。

----------
# 效果图：
![这里写图片描述](http://img.blog.csdn.net/20161205230912530)![这里写图片描述](http://img.blog.csdn.net/20161205230940046)


----------
# 实现原理：

 - DFS遍历。
 - 二维数组储存地图。
 - 


----------
# 源代码：
因为输入数据太麻烦，附了组数据。
```
#include<stdio.h>
typedef struct
{
    int x;
    int y;
    int s;
    int f;
}note;

int map[999][999], book[999][999];
note queue[999];

int main()
{
    int n, i, j, head, tail, x, y, tx, ty, flag;
    int drct[4][2] = {{0, 1}, {0, -1}, {-1, 0}, {1, 0}};

    scanf("%d", &n);

    head = 1;
    tail = 1;

    x = 1;
    y = 1;
    //scanf("%d %d", &x, &y);

    queue[tail].x = x;
    queue[tail].y = y;
    queue[tail].s = 0;
    tail++;

    for (i = 1; i <= n; i++)
    {
        for (j = 1; j <= n; j++)
        {
            scanf("%d", &map[i][j]);
        }
    }




    while (head < tail)
    {
        for (i = 0; i < 4; i++)
        {
            tx = queue[head].x + drct[i][0];
            ty = queue[head].y + drct[i][1];

            if (tx < 1 || tx > n || ty < 1 || ty > n)
            {
                continue;
            }

            if (map[tx][ty] == 0 && book[tx][ty] == 0)
            {
                book[tx][ty] = 1;

                queue[tail].x = tx;
                queue[tail].y = ty;
                queue[tail].f = head;
                queue[tail].s = queue[head].s + 1;
                //printf("###%d %d\n", tail, queue[tail].f);
                tail++;
            }

            //printf("###%d %d %d %d %d %d\n", tail, queue[tail-1].f, queue[tail - 1].x, queue[tail - 1].y, tx, ty);

            if (map[tx][ty] == 2)
            {
                flag = 1;
                break;
            }
        }

        if (flag == 1)
        {
            break;
        }
        head++;
    }

    if (flag == 1)
    {

    map[tx][ty] = 3;
    for (;;)
    {
        map[queue[head].x][queue[head].y] = 3;
        //printf("%d %d  %d\n", queue[head].x, queue[head].y, i);//
        if(head == 1)
            break;
        head = queue[head].f;
        //getchar();

    }

    putchar('\n');

    for (i = 1; i <= n; i++)
    {
        for (j = 1; j <= n; j++)
        {
            if (map[i][j] == 3)
                printf("! ");
            else if (map[i][j] == 1)
                printf("# ");
            else
                printf("* ");
        }
        putchar('\n');
    }

    }

    else
        printf("NO\n");




}

/*
20
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 1 0 1 0 0 1 0 1 0 0 1 0 1 0 0 1 0 1 0
0 1 0 0 0 0 1 0 0 1 0 0 0 0 0 0 1 0 0 0
0 0 0 0 1 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0
1 0 1 1 0 1 0 1 1 1 1 0 1 1 0 1 0 1 1 1
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 1 0 1 0 0 1 0 1 0 0 1 0 1 0 0 1 0 1 0
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
1 0 1 1 0 1 0 1 1 1 1 0 1 1 1 1 0 1 1 1
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 1 0 1 0 0 1 0 1 0 0 1 0 1 0 0 1 0 1 0
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
1 0 1 1 0 1 0 1 1 1 1 0 1 1 1 1 0 1 1 1
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 0 0
0 1 0 1 0 0 1 0 1 0 0 1 0 1 0 0 1 0 1 0
0 1 0 0 0 0 1 0 0 0 0 1 0 0 0 0 1 0 1 0
0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0
1 0 1 1 1 1 0 1 1 0 1 0 1 1 1 1 0 1 1 2
*/

```


----------
# PS:
当时刚从啊哈算法上看到DFS，还没实现过，第一次实现写的这东西，也是感兴趣，成就感满满！