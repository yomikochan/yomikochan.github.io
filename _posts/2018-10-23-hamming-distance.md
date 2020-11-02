---
title: 汉明距离(hamming distance)
layout: page
date: '2018-08-04 15:45:39 +0800'
tags:
- C/C++
- Python
categories:
- algorithms
cover_index: "/assets/28446480.jpg"
comments: true
---

### 简介

看`knn`算法时无意间发现这个算法，但是维基上有错误的示例和`python`代码。。。因为汉明距离并不是求相同长度字符串（或相同长度的整数）之间的字符（或数位）差异个数。

正确的详见：https://en.wikipedia.org/wiki/Talk:Hamming_distance

然而，我发现百度百科和一些博客都是参考的[汉明距离 - 维基百科](https://en.wikipedia.org/wiki/Hamming_distance)，所以都错了 = =。。。

认真分析正确代码后，我认为汉明距离指的是将两个字符串或两个整数编码为一组二进制数，然后计算两二进制bit之间的差异个数。

维基上有人也发现这个问题并给了一个正确的`python`代码：

```py
s1 = 'karolin'
s2 = 'kerstin'


def hamming_distance(s1, s2):
    b1, b2 = bytearray(s1, encoding='utf8'), bytearray(s2, encoding='utf8')
    diff = 0
    for i in range(len(b1)):
        if b1[i] != b2[i]:
            diff += bin(b1[i] ^ b2[i]).count("1")
    return diff


print(hamming_distance(s1, s2))
```


C/C++：

```cpp
#include <iostream>
#define ull unsigned long long
int hamming_distance(ull x, ull y)
{
	int dist = 0;
	ull or = x ^ y;
	while (or)
	{
		dist++; or &= or - 1;
	}
	return dist;
}
int main()
{
	ull x, y;
	while (std::cin >> x >> y)
		std::cout << "最小汉明距离为：" << hamming_distance(x, y) << std::endl;
	return 0;
}
```

以C代码为例，比如两个整数：1, 2，他们的二进制分别为 001, 010 ，从 001 → 010 最少需要2步替换，也就是最小汉明距离为2；又比如两整数：12, 34，他们的二进制分别为 001100, 100010，001100 → 100010 的最小汉明距离为4。

运行示例：

![run](/img/1150865.png)


注：第三个可以自己动手验证一下~

然后维基上还有一段代码：

```cpp
int hamming_distance(unsigned x, unsigned y)
{
    return __builtin_popcount(x ^ y);
}
//if your compiler supports 64-bit integers
int hamming_distance(unsigned long long x, unsigned long long y)
{
    return __builtin_popcountll(x ^ y);
}
```

好像我的电脑上不行，没有`__builtin_popcountll()`函数。
