---
title: 树状数组（Fenwick tree）
layout: page
date: '2018-08-01 21:46:03 +0800'
tags:
- C/C++
categories:
- algorithms
cover_index: "/assets/61082747.jpg"
comments: true
---

### 参考

[fenwick tree](https://en.wikipedia.org/wiki/Fenwick_tree)

### 算法实现

```C++
#include <iostream>
#define LSB(i) (i & -i)
const int M = 10001;
int bit_tree[M];
int freq[M];
int getsum(int index)
{
    int sum = 0;
    for (index += 1; index > 0; index -= LSB(index))
        sum += bit_tree[index];
    return sum;
}
void update(int n, int index, int val)
{
    for (index += 1; index <= n; index += LSB(index))
        bit_tree[index] += val;
}
void init(int freq[], int n)
{
    for (int i = 1; i <= n; i++)
        bit_tree[i] = 0;
    for (int i = 0; i < n; i++)
        update(n, i, freq[i]);
}
int main()
{
    int n;
    std::cin >> n;
    for (int i = 0; i < n; i++)
        std::cin >> freq[i];
    init(freq, n);
    for (int i = 0; i < n; i++)
    std::cout << getsum(i) << std::endl;
    for (int i = 1; i <= n; i++)
        std::cout << bit_tree[i] << ' ';
    return 0;
}
```
