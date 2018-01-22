---
title: SDUTOJ4079 - Cirno のさんすう教室(模拟)
date: 2017-11-20 12:32:20
categories: [ACM, 命题]
tags:
---
# 题目链接：

http://acm.sdut.edu.cn/onlinejudge2/index.php/Home/Contest/contestproblem/cid/2326/pid/4079

# 题目大意：

给出若干个类似 "A + B = C" 的等式（ABC均为一位的正整数），其中每个等式有可能至多有一个位置为 "?"，运算符只有"+ - *"。

题目需要输出每个复原后的等式，有可能等式本来就不需要复原，保证问号也为一位的正整数。

# 题目分析：

不难发现每个问号都有一个唯一的解，然后就是一道小学的算术题了。

分情况判断：

+ ? + B = C，那么 ? = C - B
+ A * ? = C，那么 ? = C / A
+ 以此类推....

这里可能好多同学读字符的时候出了问题，这里说一下，scanf函数里面添加一个空白字符，在读取的时候可以忽略掉若干个空白字符，比如我scanf(" %c", &ch)，就可以读取 "  1"，前者有一个空格，后者两个。


# AC代码：
```cpp
#include <stdio.h>

int main() {
    int n;
    scanf("%d", &n);
    while (n--) {
        char a, b, c, d, e;
        scanf(" %c %c %c %c %c", &a, &b, &c, &d, &e);
        a -= '0', c -= '0', e -= '0';
        if (b == '+') {
            if (a == '?' - '0') {
                a = e - c;
            } else if (c == '?' - '0') {
                c = e - a;
            } else {
                e = a + c;
            }
        } else if (b == '-') {
            if (a == '?' - '0') {
                a = e + c;
            } else if (c == '?' - '0') {
                c = a - e;
            } else {
                e = a - c;
            }
        } else {
            if (a == '?' - '0') {
                a = e / c;
            } else if (c == '?' - '0') {
                c = e / a;
            } else {
                e = a * c;
            }
        }
        printf("%c %c %c %c %c\n", a + '0', b, c + '0', d, e + '0');
    }
}
```


