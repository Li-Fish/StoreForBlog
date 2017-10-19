---
title: 使用OpenMP实现并行归并排序（Report）
date: 2017-06-05 16:41:18
categories: [CS课程]
tags:
---
# 归并排序算法：

归并排序算法是一种经典的分治算法。

## 分治
分治算法分为由三部分组成：
分解：将原问题分解为一系列子问题；
解决：递归的解决各个子问题。若子问题足够小，那么直接求解。
合并：将子问题的结果合并成原问题。

## 归并排序步骤
归并排序完全依照了上述模式，直观的操作如下：
分解：将n个元素分成各含n/2个元素的子序列；
解决：用合并排序法对两个子序列递归地排序；
合并：合并两个已经排序的子序列，已得到排序结果。
这里递归的边界是序列长度为1时，显然是有序的。


## 合并过程
这里最关键的步骤，是合并步骤里如何合并两个有序的序列，并保证合并后的序列依然有序。

假设有序的序列为递增的，A、B为需要合并的序列，C为合并后的结果序列，p、q分别为A和B的下标，top为C的下标。定义如果一个下标大于序列的长度后，表示的值为无穷大。

初始状态：p、q、top均为0.

操作：选择A[p]和B[q]中的小的元素，加入到C[top]中，然后让较小的元素所在的序列的下标加一，top加一。当A[p]和B[q]均为无穷大时，结束操作。

由于每次操作均是比较A[p]和B[q]，然后取较小者加入C中，显然时间复杂度是O(n)的。

## 归并排序时间复杂度分析：
假设归并排序一个长度为n的序列需要的时间为T(n)。
首先归并排序分如下三个步骤：
分解：这一步是把序列分为两个子序列，只需要常量时间，O(1)；
解决：递归的解决规模为n/2的两个子问题，时间为2*T(n/2)；
合并：上面已经证明，只需时间O(n)。

那么接下来可以UI递归的表示出所需的时间T(n):
当n = 1是，T(n) = O(1)；
否则：T(n) = 2*T(n/2) + O(n)。

可以证明出上述的T(n)其实就是O(n*log(n))。

T(n) = 2*T(n/2) + O(n)
		= 2*(2*T(n/4) + O(n/2) + O(n)
		= 4*T(n/4) + 2*O(n/2) + O(n)
		= 4*T(n/4) + 2*O(n)
		= 8*T(n/8) + 8*O(n/8) + 2*O(n)
		= 8*T(n/8) + 3*O(n)
		= x*T(1) + y*O(n)

显然y即为n除多少次才为1，y = log2(n)，x等于2^y，那么T(n) = O(n*log(n))。


## 一个容易理解的代码：
Python is very beautiful！
```python
def merge_sort(array):
    if len(array) > 1:
        mid = len(array) / 2
        left = merge_sort(array[:mid])
        right = merge_sort(array[mid:])
        return merge(left, right)
    return array


def merge(left, right):
    rst = []
    while len(left) > 0 or len(right) > 0:
        if len(right) == 0 or len(left) != 0 and left[0] < right[0]:
            rst.append(left.pop(0))
        else:
            rst.append(right.pop(0))
    return rst
```


# 串行过程：
## 串行排序代码：
```cpp
void merge_sort(int *A, int x, int y, int *T) {
    if (y - x > 1) {
        int m = x + (y-x)/2;
        int p = x, q = m, i = x;
        merge_sort(A, x, m, T);
        merge_sort(A, m, y, T);
        
        while (p < m || q < y) {
            if (q >= y || (p < m && A[p] <= A[q])) {
                T[i++] = A[p++];
            }
            else {
                T[i++] = A[q++];
            }
        }
        for (i = x; i < y; i++) {
            A[i] = T[i];
        }
    }
}
```

## 串行求和代码：
```cpp
int get_sum(int* data, int N) {
    int sum = 0, i;
    for (i = 0; i < N; i++) {
        sum += data[i];
    }
    return sum;
}
```

## 运行时间：

num_elements|sort time| sum time
---|------|-----------
100|0.000012|0.000001
1000|0.000166|0.000005
10000|0.002162|0.000045
100000|0.022915|0.000384
1000000|0.216075|0.003397
10000000|2.404543|0.034109
100000000|27.204318|0.340051

ignore the input time.

# 并行过程：
## 归并排序算法的并行化：

首先，归并排序的步骤分为已下三步：

分解：将n个元素分成各含n/2个元素的子序列；
解决：用合并排序法对两个子序列递归地排序；
合并：合并两个已经排序的子序列，已得到排序结果。

然后发现，按照这个思路很难并行化，因为许多过程有依赖的，比如当[1, 1], [2, 2] 区间没有合并之前，那么[1, 2], [3, 4]区间是不能进行合并的。

但是我们可以把归并的步骤反过来。原来归并是要不断的分解一个序列，直到分解成长度为1的区间，最后依次合并。我们现在假设有N个区间，要分别合并，最后合并成一个区间。那么我现在的操作是没有前后依赖的，对于任意两个区间，只需要合并就好，不用考虑其他的线程。

这样排序的过程就类似一颗线段树（严格的来讲并不是），自底向上的不断合并。

![这里写图片描述](http://img.blog.csdn.net/20170605171543250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 排序代码：
```cpp
//合并两个区间
void merge(int l1, int r1, int r2, int* data, int* temp) {
    int top = l1, p = l1, q = r1;
    while (p < r1 || q < r2) {
        if (q >= r2 || (p < r1 && data[p] <= data[q])) {
            temp[top++] = data[p++];
        }
        else {
            temp[top++] = data[q++];
        }
    }
    for (top = l1; top < r2; top++) {
        data[top] = temp[top];
    }
}

void merge_sort(int l, int r, int* data, int N) {
    int i, j, t, *temp;
    temp = (int*)malloc(N * sizeof(int));
    //这里做了一些优化，预处理合并了单个的区间，略微提高的速度
	#pragma omp parallel for private(i, t) shared(N, data)
    for (i = 0; i < N/2; i++)
        if (data[i*2] > data[i*2+1]) {
            t = data[i*2];
            data[i*2] = data[i*2+1];
            data[i*2+1] = t;
        }

	//i代表每次归并的区间长度，j代表需要归并的两个区间中最小的下标
    for (i = 2; i < r; i *= 2) {
		#pragma omp parallel for private(j) shared(r, i)
        for (j = 0; j < r-i; j += i*2) {
            merge(j, j+i, (j+i*2 < r ? j+i*2 : r), data, temp);
        }
    }
}
```

## 求和代码：
```cpp
int get_sum(int* data, int N) {
    int sum = 0, i;
    #pragma omp parallel for private(i) reduct(+:sum)
    for (i = 0; i < N; i++) {
        sum += data[i];
    }
    return sum;
}
```

## 运行时间：

num_elements|sort time| sum time
---|------|-----------
100|0.000164|0.000009
1000|0.000209|0.000009
10000|0.002318|0.000052
100000|0.010589|0.000166
1000000|0.110090|0.001279
10000000|1.093572|0.013541
100000000|11.872408|0.127646

ignore the input time.

# 运行时间分析：
## 排序时间对比：
![这里写图片描述](http://img.blog.csdn.net/20170605210453384?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 求和时间对比：
![这里写图片描述](http://img.blog.csdn.net/20170605210516264?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQUNNX0Zpc2g=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 结论：
增加线程数在是可以加快程序的运行速度的，但是随着线程的增加，加速的效果逐渐变得不明显，双线程与单线程的差异较大，整体上多线程的用时为单线程的一半。