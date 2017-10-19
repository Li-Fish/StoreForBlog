---
title: POJ2411 - Mondriaan's Dream （状压DP+轮廓线DP）
date: 2017-05-18 19:31:41
categories: [ACM, DP, 状压DP]
tags:
---
# 题目链接：
http://poj.org/problem?id=2411

----------------------
# 题目大意：
 这题题意非常明确，现在有一个 M × N 的矩形，你现在有很多个 2 × 1 大小的方块，现在要用这些方块铺满这个矩形，请问有多少种铺法。

--------------------------
# 解题过程：

 这题不是遇到卡住的，是学新知识的模板题，然后顺着书的思路做的，理解还是花了一番功夫。先看的挑战那本书，后来又翻了下大白书，还是 LRJ 的书写的详细易读，最后终于看懂了。
 
 刚开始理解错状态了，书上专门说了下多段图路径问题，然后我顺便把状态理解成每一行的对应 2 ^ M 个状态了，然后状态转移的时候怎么想都不对，最后又看了下复杂度，才发现是 O ( N × M × 2 ^ M )，然后看了下完整代码，发现每一行的每一列都是一个阶段，每个阶段对应 2 ^ M 个状态，然后一个新状态由上一个阶段的状态转移而来。

---------------------------------------
# 题目分析：

 首先确定状态。

假设我是从左上角开始依次从左至右从上至下的放方块，那么可以得出结论：假设现在要放的点为 (i, j)，大小按照字典序（先按行，后按列）。 那么对于所有的点 (i', j') >= (i, j) 一定是还没有放。对于所有 (i', j') < (i-1, j) 的点一定是已经放了方块的。


![这里写图片描述](http://img.blog.csdn.net/20170518180433429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

假设我要放置 (4, 4) 这个点，那么绿色的地方都是已经铺满的，蓝色的地方都是未铺的，黄色的地方是未确定的。接下来只要状态压缩表示黄色的部分好了。用 1 表示已铺，0 表示未铺。

![这里写图片描述](http://img.blog.csdn.net/20170518191727606?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


然后放置一个位置 (i, j)的时候有三种方式，分别是不放，向上竖着放，向左横着放。如果是不放，那么 (i-1, j) 位置一定是已经铺了的，否则不可能转移到一个合法的状态。如果是向上竖着放，那么 (i-1, j) 一定要是未铺的。如果向左横着放，那么 (i, j-1) 一定是未铺的。

![这里写图片描述](http://img.blog.csdn.net/20170518191742969?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


此外大白书上介绍了多段图的概念。有 n 列节点，每列称一个阶段，每个阶段的节点只会先下一个阶段的节点连有向边。本题就可以转化为从多段图的一个节点到达另一个节点的路径个数，矩形里的每一个方块都是一个阶段。而且递推的时候要求一个阶段只需要用到他的上一个阶段，所以可以用滚动数组实现。


# AC代码：
```cpp
#include <cstdio>
#include <cstring>
#include <vector>
#include <queue>
using namespace std;

typedef long long ll;

const int MAX = 15;
int n, m, cur;
ll dp[2][1<<MAX];

int main() {
    while (~scanf("%d %d", &n, &m) && (n+m)) {
        if (n < m) swap(n, m);
        memset(dp, 0, sizeof(dp));
        cur = 0;
        //初始化状态，看做第一行的上一行已经全部铺满
        dp[cur][(1<<m)-1] = 1;

        //枚举每一行每一列
        for (int i = 0; i < n; i ++) {
            for (int j = 0; j < m; j++) {
                //滚动数组
                cur ^= 1;
                memset(dp[cur], 0, sizeof(dp[cur]));
                for (int k = 0; k < (1<<m); k++) {
                    ll num = dp[cur^1][k];
                    //判断要放的位置的上面一块是否已铺
                    if (k&(1<<(m-1))) {
                        //不放
                        dp[cur][(k<<1)^(1<<m)] += num;
                        //判断要放的位置的左边一块是否未铺
                        if (j && !(k&1)) {
                            //向左横着放
                            dp[cur][((k<<1)^(1<<m))+3] += num;
                        }
                    }
                    else if (i) {
                        //向上竖着放
                        dp[cur][(k<<1)+1] += num;
                    }
                }
            }
        }
        printf("%lld\n", dp[cur][(1<<m)-1]);
    }
}
```