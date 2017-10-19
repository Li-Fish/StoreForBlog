---
title: HDU3085 - Nightmare Ⅱ（双向BFS）
date: 2017-05-31 20:50:32
categories: [ACM, 搜索, BFS]
tags: 
---
# 题目链接：
http://acm.hdu.edu.cn/showproblem.php?pid=3085

----------------------
# 题目大意：
 在一个N*M的网格里，有两个人 G 和 M，并且有两只鬼。
 G每秒可以走一步，G每秒可以走三步，每只鬼可以分裂，分裂到周围的的两格。假设#为鬼分裂后为1，如下图所示。每只分裂后的新鬼可以继续分裂。

0|0|1|0|0
-------|
0|1|1|1|0
1|1|#|1|1
0|1|1|1|0
0|0|1|0|0


------------------------------
# 解题过程：
 没做个两个人走的步数不同的bfs，这几天又比较咸鱼，于是去搜了题解。

http://acm.zzkun.com/archives/823

 dalao的博客。
 
------------------------
# 题目分析：
先预处理距离，这里可以了解下哈密顿距离。

然后用两个队列进行bfs，bfs操作可以写成一个函数，队列为传进来的参数，每次bfs的时候只bfs一层。层数在外面计数，传进来当参数。

inline 关键字，可以替换宏定义的函数，和宏定义的函数等价。

`Node v(1, 2)` 可以直接这样创建变量并初始化。



-----------------------------
# AC代码：
```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int x, y;
    //构造方法可以带默认值
    Node(int x = 0, int y = 0):x(x),y(y){}
};

const int dir[4][2] = {{-1,0},{0,-1},{1,0},{0,1}};
const int MAX = 800+5;

int N, M, dist[MAX][MAX];
char data[MAX][MAX];
queue<Node> q[2];
bool vis[MAX][MAX][2];
Node mm, gg, z1, z2;

void init() {
    for (int i = 1; i <= N; i++) {
        for (int j = 1; j <= M; j++) {
            //求两个幽灵到每个点的哈密顿距离，取最短的那个
            int t1 = (abs(i - z1.x) + abs(j - z1.y) + 1) / 2;
            int t2 = (abs(i - z2.x) + abs(j - z2.y) + 1) / 2;
            dist[i][j] = min(t1, t2);
        }
    }
}


//用作判断是否在边界内，这里用内联函数非常合适
inline bool in(int x, int y) {
    return x >= 1 && x <= N && y >= 1 && y <= M && data[x][y] != 'X';
}


//全局的队列，bfs函数只是用来操作一层
bool bfs(int x, int step) {
    int sz = q[x].size();
    while (sz--) {
        Node u = q[x].front(); q[x].pop();
        if (step >= dist[u.x][u.y])
            continue;
        for (int i = 0; i < 4; i++) {
            //可以直接这样调用构造方法
            Node v(u.x + dir[i][0], u.y + dir[i][1]);
            if (in(v.x, v.y) && !vis[v.x][v.y][x] && step < dist[v.x][v.y]) {
                if (vis[v.x][v.y][!x])
                    return true;
                vis[v.x][v.y][x] = true;
                q[x].push(v);
            }
        }
    }
    return false;
}

int solve() {
    while (!q[0].empty()) q[0].pop();
    while (!q[1].empty()) q[1].pop();
    q[0].push(mm), q[1].push(gg);
    memset(vis, 0, sizeof(vis));
    vis[mm.x][mm.y][0] = vis[gg.x][gg.y][1] = true;
    int step = 0;
    while (!q[0].empty() && !q[1].empty()) {
        ++step;
        //对gg bfs三层，mm bfs一层，如果相遇就返回当前的时间
        if (bfs(0, step) || bfs(0, step) || bfs(0, step) || bfs(1, step))
            return step;
    }
    return -1;
}

int main() {
    int T;
    scanf("%d", &T);
    while (T--) {
        scanf("%d %d", &N, &M);
        mm = gg = z1 = z2 = Node(0, 0);
        for (int i = 1; i <= N; i++) {
            scanf("%s", data[i]+1);
            for (int j = 1; j <= M; j++) {
                if (data[i][j] == 'G')
                    gg = Node(i, j);
                if (data[i][j] == 'M')
                    mm = Node(i, j);
                if (data[i][j] == 'Z') {
                    if (!z1.x)
                        z1 = Node(i, j);
                    else
                        z2 = Node(i, j);
                }
            }
        }
        init();
        printf("%d\n", solve());
    }
}
```