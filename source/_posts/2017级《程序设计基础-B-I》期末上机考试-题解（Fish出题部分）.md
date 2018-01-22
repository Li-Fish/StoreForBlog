---
title: 2017级《程序设计基础(B)I》期末上机考试 题解（Fish出题部分）
date: 2018-01-04 19:06:19
categories: [ACM, 命题]
tags:
---
# 喵帕斯之喵帕斯！

简单的输入输出。

```cpp
#include <stdio.h>

int main() {
int n;
    scanf("%d", &n);
    if (n == 1) printf("ohayou");
    else printf("nyanpasu");
}
```



# 喵帕斯之副食店

计算所有硬币的面额和然后除以P即可，如果大于K的话输出K。

$\min((\sum_{i = 1}^n a[i] \cdot b[i])/P, K)$就是答案。

```cpp
#include <stdio.h>

int main() {
    int n, k, p;
    while (~scanf("%d %d %d", &n, &k, &p)) {
        int sum = 0;
        for (int i = 1; i <= n; i++) {
            int a, b;
            scanf("%d %d", &a, &b);
            sum += a * b;
        }
        int ans = sum / p;
        if (ans > k) ans = k;
        printf("%d\n", ans);
    }
}
```

# 喵帕斯之天才算数少女

考察函数的递归应用，根据题意写出代码即可。

（其实这个函数是理论计算机科学里非常重要的一个函数，阿克曼函数）

```cpp
#include <stdio.h>

int fun(int m, int n) {
    if (m == 0) return n + 1;
    else if (m > 0 && n == 0) return fun(m - 1, 1);
    else return fun(m - 1, fun(m, n - 1));
}

int main() {
    int m, n;
    while (~scanf("%d %d", &m, &n)) {
        printf("%d\n", fun(m, n));
    }
}
```

# 喵帕斯之传说中的神剑

![](http://imgsrc.baidu.com/forum/pic/item/9178281f95cad1c8a2e3cbea7a3e6709c83d515c.jpg)

其实喵帕斯参加过圣杯战争！（雾

这个题就是一个简单的打印图形，没什么难点，对比打印金字塔和菱形简单多了。注意点格式不要PE就好了。

```cpp
#include <stdio.h>

int main() {
    int a, b;
    while (~scanf("%d %d", &a, &b)) {
        for (int i = 1; i <= a; i++) {
            if (i == a / 3 * 2) {
                for (int i = 1; i <= b; i++) {
                    printf("#");
                }
            } else {
                for (int i = 1; i <= b / 2; i++) {
                    printf(" ");
                }
                printf("#");
            }
            putchar('\n');
        }
        putchar('\n');
    }
}
```

# 喵帕斯之数学难题

考察拆分整数和素数判断，其实直接读字符串也可以。推荐写个判断是否伟素数的函数。

```cpp
#include <stdio.h>

int is_prime(int n) {
    if (n == 1) return 0;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) return 0;
    }
    return 1;
}

int main() {
    int n;
    while (~scanf("%d", &n)) {
        int sum = 0;
        while (n) {
            sum += (n % 10) * (n % 10);
            n /= 10;
        }
        if (is_prime(sum)) printf("YES\n");
        else printf("NO\n");
    }
}
```

# 喵帕斯之平地摔

简单的一位数组，判断下是否大于相邻的两个位置即可。

```cpp
#include <stdio.h>

#define MAX 233

int data[MAX];

int main() {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%d", data + i);
        }
        int ans = 0;
        for (int i = 2; i <= n - 1; i++) {
            if (data[i] > data[i - 1] && data[i] > data[i + 1]) ans++;
        }
        printf("%d\n", ans);
    }
}
```

# 喵帕斯之短笛

考察字符串比较，注意最后一个数后面没有空格。

```cpp
#include <stdio.h>
#include <string.h>

char str[][3] = {"do", "re", "mi", "fa", "so", "la", "xi"};

char data[3];

int find() {
    int pos = 0;
    while (strcmp(data, str[pos]) != 0) pos++;
    return pos + 1;
}

int main() {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            scanf("%s", data);
            printf("%d%c", find(), i == n ? '\n' : ' ');
        }
    }
}
```

# 喵帕斯之矩阵

算是最难的题了，考察排序和二维数组，排序的时候需要找一下下标之间的关系。

这道题可以有多个做法，这里推荐一下郑康的做法。

让每个位置的数和他右下角的去比较，因为最长的对角线最长为 N，跑 N - 1 趟就OK。

大家可以类比下冒泡去理解一下。

```cpp
#include <stdio.h>
#define MAX 112

int data[MAX][MAX];

int main() {
    int n;
    while (~scanf("%d", &n)) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                scanf("%d", &data[i][j]);
            }
        }
        for (int k = 1; k < n; k++) {
            for (int i = 1; i < n; i++) {
                for (int j = 1; j < n; j++) {
                    if (data[i][j] > data[i + 1][j + 1]) {
                        int t = data[i][j];
                        data[i][j] = data[i + 1][j + 1];
                        data[i + 1][j + 1] = t;
                    }
                }
            }
        }
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                printf("%d%c", data[i][j], j == n ? '\n' : ' ');
            }
        }
    }
}
```

