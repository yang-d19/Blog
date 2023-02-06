---
title: Codeforces 题解 1553B [Reverse String]
typora-root-url: ./
categories: [技术]
tags: [算法, Codeforces]
date: 2021-08-18
---

**【题目大意】** 给定字符串 s，给出一个由 s 生成新字符串 t 的方法 M：将指针 p 放在 s 的某一位上，然后将 p 向右移动若干次，再向左移动若干次（p 始终指向 s 中的字符），p 指到的所有字符排列成一个新字符串 t。例如 s 为 abcdef，p 初始为 2（即指向 c），向右移动 2 次，向左移动 3 次，生成的 t 为 cdedcb。现给出若干组 s 和 t，判断 t 能否由 s 使用 M 方法生成。

**【数据范围】** 组数 $q \le 500$， $|s| \le 500$，$sum(|s|) \le 500$

<!--more-->

## Solution 1

### Idea

遍历 s 中的所有折返点，对于每个折返点都生成一个新串 s’，在 s’ 中查找是否存在子串 t，若所有的 s’ 中均找不到 t，则失败；否则成功。

### Complexity

生成一个 s’ 的复杂度是 $O(N)$，在一个 s’ 中查找 t 的复杂度是 $O(N^2)$，总共有 N 个 s’，所以总复杂度为 $O(N^4)$。

注：若不主动生成每个 s’，而是遍历所有起始点和折返点手动判断字符串是否相等，则复杂度为 $O(N^3)$

### Code

```c++
#include <iostream>
#include <string>
#include <algorithm>

using namespace std;

string extStr(string s, int rev_pt)
{
    string part1 = s.substr(0, rev_pt);
    string part2 = s.substr(rev_pt, 1);

    string part3 = part1;
    reverse(part3.begin(), part3.end());

    return part1 + part2 + part3;
}

bool getAns(string a, string b)
{
    for (int rev_pt = 0; rev_pt < a.size(); rev_pt++)
    {
        string ext = extStr(a, rev_pt);

        if (ext.find(b) != string::npos)
            return true;
    }
    return false;
}

int main(void)
{
    int N;
    string a, b;
    string ans;
    
    cin >> N;

    for (int count = 0; count < N; count++)
    {
        cin >> a >> b;
        getAns(a, b) ? cout << "YES" : cout << "NO";
        cout << endl;
    }
    return 0;
}
```

## Solution 2

### Idea

使用字符串哈希的思想，不再需要使用 string.find() 函数。遍历生成字符串 t’ 的起始点 i 和反转点 j，t 的长度已知时，终点 k 于是也已知。比对 t’ 和 t 的哈希值，可知 t 是否可由 s 生成。

h[i] 和 r[i] 中分别存储 s[0~i] 和 s[n~i] 的哈希值。hash(s[i~j~k]) 可由 h[i], h[j], r[j], r[k] 运算得到，不需要重复运算。

### Complexity

预处理 h 和 r 数组：$O(N)$

计算 hash(s[i~j~k])：$O(1)$

遍历所有的 i, j：$O(N^2)$

所以总复杂度为 $O(N^2)$

### Code

```c++
#include <iostream>
#include <string>
#include <algorithm>
#include <iomanip>
#include <cmath>
#include <cstdio>

using namespace std;

const long long MOD = 50331653;
const int LEN = 510;

long long h[LEN], r[LEN];
long long pow26[LEN];

long long CalcHash(int x, int y, int len);
long long CalcHash(string s);
bool GetAns(string s, string t);

int main(void)
{
    int Q;
    string s, t;
    
    cin >> Q;

    pow26[0] = 1;
    for (int i = 1; i < LEN; i++)
    {
        pow26[i] = (pow26[i - 1] * 26) % MOD;
    }

    while (Q--)
    {
        cin >> s >> t;
        printf("%s\n", GetAns(s, t) ? "YES" : "NO");
    }
    return 0;
}

bool GetAns(string s, string t)
{
    long long hash = 0;
    for (int i = 0; i < s.size(); i++)
    {
        hash = (hash * 26 + s[i] - 'a') % MOD;
        h[i] = hash;
    }
    hash = 0;
    for (int j = s.size() - 1; j >= 0; j--)
    {
        hash = (hash * 26 + s[j] - 'a') % MOD;
        r[j] = hash;
    }

    long long target = CalcHash(t);
    int len = t.size();
    for (int x = 0; x < s.size(); x++)
    {
        int low = ceil((x + len - 1) / 2.0);
        int high = len + x -1;
        for (int y = low; y <= high; y++)
        {
            long long tmp = CalcHash(x, y, len);
            if (target == tmp)
            {
                return true;
            }
        }
    }
    return false;
}

long long CalcHash(int x, int y, int len)
{
    int z = 2 * y - x + 1 - len;

    long long hash1;
    if (x > 0)
        hash1 = ((h[y] - h[x - 1] * pow26[y - x + 1]) % MOD + MOD) % MOD;
    else
        hash1 = h[y] % MOD;

    long long hash2 = ((r[z] - r[y] * pow26[y - z]) % MOD + MOD) % MOD;
    long long hash = (hash1 * pow26[y - z] + hash2) % MOD;

    return hash;
}

long long CalcHash(string s)
{
    long long hash = 0;
    for (int i = 0; i < s.size(); i++)
    {
        hash = (hash * 26 + s[i] - 'a') % MOD;
    }
    return hash;
}
```
