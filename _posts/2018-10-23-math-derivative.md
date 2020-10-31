---
title: 数学-导数篇（持续更新）
layout: page
date: '2018-09-05 19:39:45 +0800'
tags:
- Math
categories:
- mathematical
cover_index: "/assets/472062875.jpg"
math: true
---

### 简介

这里主要存放一些导数题

### 题

1.已知向量 $ a = (sin(x), \frac{3}{4}), b = (cos(x), -1) $.

(1) 当 $ a // b $ 时，求 $ cos^2(x) - sin(2x) $;

(2) 设函数 $ f(x) = 2(\vec{a} + \vec{b}) \cdot \vec{b} $, 已知在△ABC中，内角A、B、C的对边分别为 $ a, b, c $,若 $ a = \sqrt{3}, b = 2, sin(B) = \frac{\sqrt{6}}{3} $,求 $ f(x) + 4 cos(2A + \frac{\pi}{6})(x \in [0, \frac{\pi}{3}]) $ 的取值范围。

题解：

这题虽然是一张向量试卷上的，但第二问基本上用导数就可以了，而且，我不知道题目给的几个边的条件有什么用...

(1) 求 $ cos^2(x) - sin(2x) $

由 $ a // b $，有： $ tan(x) = -\frac{3}{4} $.

对 $ cos^2(x) - sin(2x) $ 做一些变形：

$$
\begin{aligned}
cos^2(x) - sin(2x)
&= cos^2(x)(1 - \frac{sin(2x)}{cos^2(x)}) \\
&= cos^2(x)(1 - \frac{2sin(x)cos(x)}{cos^2(x)}) \\
&= cos^2(x)(1 - 2tan(x))
\end{aligned}
$$

然后考虑 $  tan(x) = -\frac{3}{4} = \frac{sin(x)}{cos(x)} $

有以下情况：

1. $ \sqrt{(-3)^2 + 4^2} = 5 $ 或 $ \sqrt{3^2 + (-4)^2} = 5 $

2. $ sin(x) $ 为负或者 $ cos(x) $ 为负

既有 $ cos(x) = -\frac{4}{5} $ 或者 $ cos(x) = \frac{4}{5} $

这并不影响到 $ cos^2(x) $ 的值，所以有：

$$
\begin{aligned}
cos^2(x) - sin(2x)
&= cos^2(x)(1 - 2tan(x)) \\
&= \frac{16}{25} \cdot [1 - 2 \cdot (-\frac{3}{4})] \\
&= \frac{8}{5}
\end{aligned}
$$

(2) 求 $ f(x) + 4cos(2A + \frac{\pi}{6}) (x \in [0, \frac{\pi}{3}]) 的取值范围 $

令 $ \varphi (x) = f(x) + 4cos(2A + \frac{\pi}{6}) (x \in [0, \frac{\pi}{3}]) $

由题目条件知： $ f(x) = 2cos^2(x) + sin(2x) + \frac{1}{2} $

$$
\begin{aligned}
f'(x)
&= -4cos(x)sin(x) + 2cos(2x) \\
&= 2(cos(2x) - sin(2x)) \\
&= 2\sqrt{2}sin(\frac{\pi}{4} - 2x)
\end{aligned}
$$

令 $ f'(x) = 0 $, 则可得到 $ x = \frac{\pi}{8} + 2k\pi (k = 0,1,2...) $

$ \because x \in [0, \frac{\pi}{3}], \therefore x = \frac{\pi}{8} $

又

① $ f'(0) > 0; $
② $ f'(\frac{\pi}{3}) < 0. $

考虑到 $ f'(x) $ 是一个最小周期为 $ \pi $ 的函数，所以 $ f'(x) $ 在 $ x \in [0, \frac{\pi}{3}] $ 内要么单调递增，要么单调递减，所以有如下分析：

① $ f'(x) $ 在 $ x \in [0, \frac{\pi}{3}) $ 恒大于 0，则原函数 $ f(x) $ 在 $ x \in [0, \frac{\pi}{3}) $ 内单调递增;
② $ f'(x) $ 在 $ x \in (\frac{\pi}{8}, \frac{\pi}{3}] $ 恒小于 0，则原函数 $ f(x) $ 在 $ x \in (\frac{\pi}{8}, \frac{\pi}{3}] $ 内单调递减.

所以 $ x = \frac{\pi}{8} $ 为 $ f(x) $ 在 $ x \in [0, \frac{\pi}{3}) $ 内的唯一一个最大值.

又 $ f(0) > f(\frac{\pi}{3}) $ ，所以，$ f(\frac{\pi}{3}) $ 是 $ f(x) $ 在 $ x \in [0, \frac{\pi}{3}) $ 内的唯一一个最小值.

可以借助工具来画出函数图看一看是否正确：

![图1](/img/2cos^2x+sin2x+1_2.png)

$ f(\frac{\pi}{8}) = \frac{3}{2} + \sqrt{2}, f(\frac{\pi}{3}) = 1 + \frac{\sqrt{3}}{2} $

于是

$$
\begin{aligned}
max \ \varphi(x) &= f(\frac{\pi}{8}) + 4 = \frac{11}{2} + \sqrt{2} \\
min \ \varphi(x) &= f(\frac{\pi}{3}) - 4 = \frac{\sqrt{3}}{2} - 3
\end{aligned}
$$

2.
