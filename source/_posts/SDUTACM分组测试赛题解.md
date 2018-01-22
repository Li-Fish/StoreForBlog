---
title: SDUTACM分组测试赛题解
date: 2018-01-18 18:38:41
categories: [命题]
tags:
---
# A - China Final

简单的模拟计算 ACM-ICPC 的排名。

````cpp
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n;
    while(scanf("%d",&n)!=EOF) {
        printf("%d %d %d\n",n/10,n/10*3,n/10*6);
    }
    return 0;
}
````

# B - 为了相同的前缀-跳楼梯

判断一下是否有两个或多个相邻的台阶同时脏了，或者第 100 个台阶是否脏了即可。

```cpp
#include <stdio.h>

#define MAX 200
int data[MAX];

int main() {
    int n;
    while (~scanf("%d", &n)) {
        int flag = 1;

        for (int i = 1; i <= n; i++) {
            scanf("%d", data + i);
            if (data[i] == 100) flag = 0;
        }

        for (int i = 1; i <= n - 1; i++) {
            for (int j = 1; j < n - i + 1; j++) {
                if (data[j] > data[j + 1]) {
                    int t = data[j + 1];
                    data[j + 1] = data[j];
                    data[j] = t;
                }
            }
        }

        for (int i = 2; i <= n; i++) {
            if (data[i] > 100) break;
            if (data[i] - data[i - 1] == 1) flag = 0;
        }

        if (flag) printf("Orz\n");
        else printf("Why are you so ben?\n");
    }
}
```

# C - 完美区间

写好判断素数的函数，然后跑一个 for 循环即可，注意 0 和 1 不是素数。

```cpp
#include <stdio.h>

int is_prime(int n) {
    if (n <= 1) return 0;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) return 0;
    }
    return 1;
}

int main() {
    int l, r;
    while (~scanf("%d %d", &l, &r)) {
        int cnt = 0;
        for (int i = l; i <= r; i++) {
            if (is_prime(i)) cnt++;
        }
        if (is_prime(cnt)) printf("Yes\n");
        else printf("No\n");
    }
}
```

# D - 2017 666!

直接当做字符串读入即可，然后 if 判断有几个 2017 。

```cpp
#include <stdio.h>
#include <string.h>

#define MAX 10

char data[MAX];

int main() {
    int l, r;
    while (~scanf("%s", data)) {
        int len = strlen(data);
        int ans = 0;
        for (int i = 0; i + 3 < len; i++) {
            if (data[i] == '2' &&
                data[i + 1] == '0' &&
                data[i + 2] == '1' &&
                data[i + 3] == '7')
                ans++;
        }
        while (ans--) {
            printf("666");
        }
        printf("\n");
    }
}
```

# E - 场地安排

可以先把数组初始化为 -1，并且最小的下标从  (1, 1) 开始，那么就不需要特判边界了。

对于每个位置的相邻位置可以把增量算出来存到数组里面，比如当前位置是第 i 行第 j 列，那么这个的左上角就是 (i - 1, j - 1)，记录增量 (-1, -1)。



```cpp
#include <stdio.h>
#include <string.h>

#define MAX 112

int data[MAX][MAX];
int dirc[8][2] = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}, {1, 1}, {1, -1}, {-1, 1}, {-1, -1}};

int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m)) {
        memset(data, -1, sizeof(data));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                scanf("%d", &data[i][j]);
            }
        }
        int flag = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                for (int k = 0; k < 8; k++) {
                    int tx = i + dirc[k][0];
                    int ty = j + dirc[k][1];
                    if (data[tx][ty] == data[i][j]) flag = 0;
                }
            }
        }
        if (flag) printf("\\(^o^)/~\n");
        else printf("Oh~no!!!\n");
    }
}
```

# F - 及格名单

将名字和成绩用数组存起来，最后输入及格线，然后在 for 循环一遍数组，如果及格就输出。

```cpp
#include <stdio.h>

char name[101][20];
int score[101];

int main() {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            int t;
            scanf("%s %d", name[i], score + i);
        }
        int bound;
        scanf("%d", &bound);
        for (int i = 1; i <= n; i++) {
            if (score[i] >= bound) printf("%s %d\n", name[i], score[i]);
        }
    }
}
```

# G - 小鑫与斐波那契（二）

有个定理$(a + b) \mod p = (a \mod p + b \mod p) \mod p$。

然后计算过程中可能会有的项超出 long long 的范围。根据上面的定理显然可以知道总结取膜对结果无影响，然后 for 循环计算即可。

注意直接用函数递归一定会超时，因为计算量几乎是按指数递增的，可以简单的画图看一下，有很多项被重复计算了多次。

![](http://chuantu.biz/t6/209/1516275572x-1404793130.png)

```cpp
#include <stdio.h>

#define MAX 1000123
#define MOD 1000000007

long long fib[MAX];

int main() {
    int n;
    fib[0] = 0, fib[1] = 1;
    for (int i = 2; i < MAX; i++) {
        fib[i] = (fib[i - 1] + fib[i - 2]) % MOD;
    }
    while (~scanf("%d", &n)) {
        printf("%lld\n", fib[n]);
    }
}
```

# H - 今年暑假不AC

贪心的经典例题，按每个时间段的右边界排序，然后贪心的尽可能的选就可以了。

（如果不会的话可以等到贪心的专题，肯定会有学长 or 老师讲这道题）

```cpp
#include <stdio.h>

#define MAX 112

struct {
    int l, r;
} data[MAX], t;

int main() {
    int n;
    while (scanf("%d", &n) && n) {
        for (int i = 1; i <= n; i++) {
            scanf("%d %d", &data[i].l, &data[i].r);
        }
        for (int i = 1; i <= n - 1; i++) {
            for (int j = 1; j < n - i + 1; j++) {
                if (data[j].r > data[j + 1].r) {
                    t = data[j + 1];
                    data[j + 1] = data[j];
                    data[j] = t;
                }
            }
        }
        int pos = 0, cnt = 0;
        for (int i = 1; i <= n; i++) {
            if (pos <= data[i].l) pos = data[i].r, cnt++;
        }
        printf("%d\n", cnt);
    }
}
```

# I - 数字矩阵

设 $dp[i][j]$ 为走到 $dp[i][j]$ 的最大累加和，$data[i][j]$ 为 i 行 j 列的值，那么从左上到右下有：

$$dp[i][j] = \max(dp[i - 1][j], dp[i][j -1]) + data[i]$$

对于右上到左下有：

$$dp[i][j] = \max(dp[i - 1][j], dp[i][j + 1]) + data[i]$$

这里虽然有四个起始位置，但是有两个是重复的只是起点和终点交换了，跑两次 DP 即可。

```cpp
#include <stdio.h>
#include <string.h>

#define MAX 200

int max(int a, int b) {
    return a > b ? a : b;
}

int a[MAX][MAX], b[MAX][MAX], c[MAX][MAX];

int main() {
    int n, m;
    while (~scanf("%d %d", &n, &m)) {
        memset(a, 0, sizeof(a));
        memset(b, 0, sizeof(b));
        memset(c, 0, sizeof(c));
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                scanf("%d", &a[i][j]);
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                b[i][j] = a[i][j] + max(b[i - 1][j], b[i][j - 1]);
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = m; j >= 1; j--) {
                c[i][j] = a[i][j] + max(c[i - 1][j], c[i][j + 1]);
            }
        }
        printf("%d\n", max(b[n][m], c[n][1]));
    }
    return 0;
}
```

