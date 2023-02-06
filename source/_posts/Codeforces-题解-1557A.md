---
title: Codeforces 题解 1557A [Ezzat and Two Subsequences]
typora-root-url: ./
categories: [技术]
tags: [算法, Codeforces]
date: 2021-08-18
---

**【题目大意】** 给定 n 个数字组成的数组（可为负），将其分为两组非空的子数组 a, b，定义 f(x) 为数组 x 的平均值，求 f(a) + f(b) 的最大值。

**【数据范围】** 组数 $t \le 10^3$， $n \le 10^5$，$|a_i|\le 10^9$，$sum(n)\le 3\cdot 10^5$

<!--more-->

## Solution

### Idea

乍一看感觉是道有点难度的数学题，但是放在 A 的位置，肯定没有那么复杂。题目中给了不少的示例，通过观察示例可以猜到，应该是将数组中最大的一个元素单独拿出来成为一组，另外 n-1 个组成一组，这样的结果应该是最大的。

比赛的时候我用 double ，输出保留 9 位小数结果居然错了，怀疑是思路有问题，一直空到最后，快截止的时候发现整形数据我使用的是 int ，改成了 long long 之后提交通过了。

数学证明如下（翻译自 Codeforces 官方 tutorial）

首先有：一组数的均值总是大于等于这组数中的最小值，小于等于这组数中的最大值。

将数组排列有序：$a_1 \le a_2 \le … \le a_n$ ，下面用反证法证明：

假设当选取最大的两个数字组成一组时的结果比只选取最大的一个数组时的结果更大，即
$$
\frac{\sum_{i=1}^{n-1}a_n}{n-1} + a_n < \frac{\sum_{i=1}^{n-2}a_i}{n-2}+\frac{a_{n-1}+a_n}{2}
$$
化简得
$$
a_n < \frac{2\sum_{i=1}^{n-2}a_i}{(n-1)(n-2)}+\frac{n-3}{n-1}a_{n-1}
$$
记 $avg_1 = \sum_{i=1}^{n-2}a_i/(n-2)$，因为 $a_n$ 有序，所以 $a_1 \le avg_1 \le a_{n-2}$

所以上式可以化为
$$
a_n < \frac{2\cdot avg_1 + (n-3)a_{n-1}}{n-1}
$$
不等式右边可以看做 $[avg_1, avg_1, a_{n-1}, a_{n-1}, a_{n-1}…]$ 的平均值，记为 $avg_2$，可知 $avg_1 \le avg_2 \le a_{n-1}$

因此
$$
a_n < avg_2 \le a_{n-1} \le a_n
$$
矛盾，证毕

不过官方题解只证明了问题的一小部分，并不是原问题的完美解答，具体证明我也不会，不过比赛的时候只管猜就是了。

### Complexity

$O(N)$ 不解释

### Code

```c++
#include <iostream>
#include <algorithm>
#include <cstdio>
using namespace std;

int T, N;

int main()
{
    cin >> T;
    while (T--)
    {
        cin >> N;
        long long sum = 0;
        long long max_num = -1000000001;
        long long num;
        double ans;
        for (int i = 0; i < N; i++)
        {
            cin >> num;
            sum += num;
            max_num = max(max_num, num);
        }
        ans = (double) (sum + (N - 2) * max_num) / (double) (N - 1);
        printf("%.9lf\n", ans);
    }
    return 0;
}
```
