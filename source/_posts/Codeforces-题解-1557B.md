---
title: Codeforces 题解 1557B [Moamen and k-subarrays]
typora-root-url: ./
categories: [技术]
tags: [算法, Codeforces]
date: 2021-08-18
---

**【题目大意】** 给出 n 个数组成的数组 a，记操作 M 为：将 a 切分为 k 份，这 k 份任意排列，组成序列 b。若存在某种操作 M，使得 b 单调不减，则输出 Yes，否则 No

**【数据范围】** 组数 $t \le 10^3$，$k \le n \le 10^5$，$|a_i| \le 10^9$ 且 $a_i$ 互不相等，$\sum n \le 3\cdot 10^5$

<!--more-->

## Solution

### Idea

要使得 k 组子序列排列后构成单增序列，则每个子序列的内部是单调增且紧密的，“紧密”指的是不可能从该子序列的外部找到一个这样一个元素：该元素可以插入该子序列内部，并维持序列的单调性。例如，若将 [3, 5, 6, 1, 2, 4] 分割为 [3, 5, 6] 和 [1, 2, 4]，虽然每个子序列是单增的，但是 3 可以插入到 [1, 2, 4] 中组成 [1, 2, 3, 4]，因此排列这两个子序列无论如何也不能得到一个单增的完整序列。

构造结构体 s，存储每个元素的值 val 和其在原序列中的序号 idx，按照 val 对 s 从小到大排序，依次遍历 s 中元素，若 val 顺序与 idx 顺序保持相邻，则表示可以切分入同一个子序列中，否则增加 section 值，表示需要另开一个子序列。

最后比较 section 和 k 的大小。

### Complexity

快排：$O(N\cdot logN)$

遍历：$O(N)$

总复杂度：$O(N\cdot logN)$

### Code

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int T, N, K;

struct coup
{
    int num;
    int idx;
}s[100010];

bool cmp(coup a, coup b)
{
    return a.num < b.num;
}

int main(void)
{
    cin >> T;
    while (T--)
    {
        cin >> N >> K;
        int tmp;
        for (int i = 0; i < N; i++)
        {
            cin >> tmp;
            s[i].num = tmp;
            s[i].idx = i;
        }
        sort(s, s + N, cmp);
        int section = 0;
        int cnt = 0;
        while (cnt < N)
        {
            while (cnt + 1 < N && s[cnt + 1].idx == s[cnt].idx + 1)
            {
                cnt++;
            }
            cnt++;
            section++;
        }
        if (section <= K)
            cout << "Yes" << endl;
        else 
            cout << "No" << endl;
    }
    return 0;
}
```
