---
title: 计算行列式（高斯消元？+Java+工具）
date: 2017-03-09 15:06:33
categories: [编程语言, Java]
tags:
---
# 功能：
-----------------
#### 计算行列式并输出
# 用法：
-------------------------
首先输入行列式的阶数，然后以输入行列式内容。

例如：

输入：
4
1 2 -1 3
2 3 -1 2
-1 1 1 0
0 1 -2 1

输入：
18.0
# 实现：
--------------------
好像是高斯消元，就是每一行乘一个系数减下去，化三角。
时间复杂度 O(n^3) .
# 代码：
```cpp
import java.util.*;
import java.lang.*;

public class Determinant {

    public static void main(String[] Args){
        Scanner cin = new Scanner(System.in);
        int size = cin.nextInt();
        double[][] data = new double[size][size];

        //读取数据
        for (int i = 0; i < size; i++) {
            for (int j = 0; j < size; j++) {
                data[i][j] = cin.nextDouble();
            }
        }

        //标记是否可以提前确定行列式为0
        boolean zero = false;
        //记录交换行后的正负号
        double sign = 1;

        //主体
        for (int row = 0; row < size-1; row++){

            //如果第i行第i个元素已经是0的话，就去找一行非零的行交换
            if (data[row][row] == 0) {
                boolean flag = false;
                for (int trow = row + 1; trow < size; trow++){
                    if (data[trow][row] != 0){
                        flag = true;
                        swap_row(row, trow, size, data);
                        sign *= -1;
                        break;
                    }
                }
                //如果一列全为0，那么行列式为0
                if (!flag)
                {
                    zero = true;
                    break;
                }
            }

            for (int trow = row + 1; trow < size; trow++){
                double k;
                //如果当前行已经为0，那么直接跳过，否则用row行乘-k加到trow行
                if (data[trow][row] == 0)
                    continue;
                k = data[trow][row] / data[row][row];
                for (int col = row; col < size; col++){
                    data[trow][col] += -k * data[row][col];
                }
            }
        }

        double ans;


        //整理答案并输出
        if (zero)
            ans = 0;
        else{
            ans = sign;
            for (int row = 0; row < size; row++){
                ans *= data[row][row];
            }
        }

        System.out.print(ans);
    }

    static void swap_row(int a, int b, int size, double data[][]){
        for (int i = a; i < size; i++){
            double temp = data[a][i];
            data[a][i] = data[b][i];
            data[b][i] = temp;
        }
    }
}


```