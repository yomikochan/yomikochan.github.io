---
layout: page
title: 梯度下降算法
date: 2018-07-26 23:30:49+8
tags:
  - Math
  - C/C++
  - MatLab
  - Python
categories: algorithms
cover_index: /assets/60572969.jpg
---

### 补充说明

代码更新了，为了解释为什么代码这样写，先来说说梯度下降法的迭代公式。

首先，暂时只考虑二元函数 $ f(x, y) $，因为在能可视化的三维图下研究梯度更好理解，但在文章中形式上我还是把它考虑写为 $ f(\vec x) $。

设函数为 $ f(\vec x) = ag(x_1) + b\phi{(x_2)} + c $

函数从某个起点 $ \vec{p}_{(1)} = (p_1, p_2) $ 处出发，通过求梯度进行方向搜索，找到某个步长范围内下降最快的方向的点，然后从初始点移动到找到的点。

迭代公式：

$$ \vec{p_{n + 1}} = \vec{p_{n}} + \lambda \dot (-\bigtriangledown f(\vec x)) $$

这里 $ \lambda $ 是一个步长系数，我们可以自己控制它的增长，它的作用就在于有时候如果步长设置过大，可能就会跳过我们要走的方向，过小可能完全找不到方向，所以还有更好的自动计算方式：

我们的迭代公式求出来的是一个点，于是将 $ \vec{x{n}} + \bigtriangledown f(\vec x) \dot \lambda $ 代入原函数中，得到：

$$ \omega (\lambda)) = ag(p_1 + \lambda \dot (-\bigtriangledown f(\vec{p}_{(1)}))) + b\phi{(p_2 + \lambda \dot (-\bigtriangledown f(\vec{p}_{(1)})))} $$

令 $ \omega '(\lambda) =  0 $ ，整理得：

$$ a^2 g'(p_1) g'(p_1 - a\lambda g'(p_1)) + b^2 \phi '(p_2) \phi '(p_2 - b\lambda \phi '(p_2)) = 0 $$

因为参数a,b和点(p_1, p2)都已知，所以可求出 $ \lambda $ 的值，这就是我代码中`ployeq()`函数的意义。

本来为了便利性，所以利用回调函数`gradient_eq`来方便自己定制方程的梯度公式，做了种种尝试后发现，不会符号计算还是不能便利的求`ployeq()`函数，因为λ是一个未知变量，不好代公式求出来。。。所以，想求方程，还是只有麻烦自己先手动推出计算公式啦~

### 关于算法

[Gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) 又叫最速下降法，是一种局部搜索算法，属于智能算法，类似的有爬山算法、模拟退火等。基本原理是优化理论中的无约束问题，利用梯度搜索函数变化最快的方向，然后不断迭代的方法。

好久之前学过这个算法，但当时没实际写过代码，仅仅推过公式，然后做了一道题。。。后来一直没用上，就忘了一些推导过程了。。。

今天有点晚，暂时就不详细推公式了，有空我将补上推导过程并给出梯度法的收敛条件，还会给出一些更好的优化算法(牛顿法、共轭梯度法等)及其他约束问题的优化算法，届时我们将会看到许多有趣的搜索算法，比如斐波那契法、黄金比例法(0.618法)等。

### 代码实现

由于咱太蒻稽...不会符号计算，不能写一个通用梯度下降算法模板。。。于是这里以问题
$$ min \ f(x) = 2x_1 ^2 + x_2 ^2, $$
起始点 $ x_0 = (1, 1)^T, \epsilon = \frac{1}{1000} $ 为例。

函数图：

![f(x) = 2x_1 ^2 + x_2 ^2](/img/funcgraph.png)

算法实现：

##### C programming language

```C
#include <stdio.h>
#include <math.h>
#define square(x) x*x
typedef double(*gradient_eq)(double, ...);
double partial_x(double x, ...)
{
    return 4*(-x);
}
double partial_y(double x, ...)
{
    return 2*(-x);
}
double hypotenuse(double a, double b)
{
    return sqrt(square(a) + square(b));
}
double ployeq(double x_1, double x_2, double a, double b)
{
    return -(2*a*x_1 + b*x_2)/(2*square(a) + square(b));
}
void gradient_descent(
    double x_1,
    double x_2,
    gradient_eq grad_x,
    gradient_eq grad_y,
    double epsilon,
    unsigned max_iters)
{
    double a, b, lambda;
    unsigned iters = 0;
    do
    {
        printf ("(%f, %f)\n", x_1, x_2);
        a = grad_x(x_1), b = grad_y(x_2);
        lambda = ployeq(x_1, x_2, a, b);
        x_1 += a*lambda;
        x_2 += b*lambda;
        iters++;
    } while (hypotenuse(a, b) > epsilon && iters < max_iters);
}
int main()
{
    double x_1 = 1., x_2 = 1.;
    gradient_descent(x_1, x_2, partial_x, partial_y, 1e-3, 100);
    return 0;
}
```


Output:

$ \epsilon = 10^-3 $

```bash
(1.000000, 1.000000)
(-0.111111, 0.444444)
(0.074074, 0.074074)
(-0.008230, 0.032922)
(0.005487, 0.005487)
(-0.000610, 0.002439)
(0.000406, 0.000406)
(-0.000045, 0.000181)
```

$ \epsilon = 10^-6 $

```
(1.000000, 1.000000)
(-0.111111, 0.444444)
(0.074074, 0.074074)
(-0.008230, 0.032922)
(0.005487, 0.005487)
(-0.000610, 0.002439)
(0.000406, 0.000406)
(-0.000045, 0.000181)
(0.000030, 0.000030)
(-0.000003, 0.000013)
(0.000002, 0.000002)
(-0.000000, 0.000001)
(0.000000, 0.000000)


问题最优解：(0, 0)
```

##### Python3

```python
square = lambda x: x ** 2
side = lambda a, b: (a ** 2 + b ** 2) ** (1 / 2)


def d(x_1, x_2):
    return -4 * x_1, -2 * x_2

def evaluate(x_1, x_2, a, b):
    return -(2 * a * x_1 + b * x_2) / (2 * square(a) + square(b))

def gradient_descent(x_1, x_2, epsilon, max_iters):
    iters = 0
    a, b = 1, 1
    while (side(a, b) > epsilon) & (iters < max_iters):
        print('%d (%.6f, %.6f)' % (iters + 1, x_1, x_2))
        a, b = d(x_1, x_2)
        gamma = evaluate(x_1, x_2, a, b)
        (x_1, x_2) = (x_1 + a * gamma, x_2 + b * gamma)
        iters += 1


if __name__ == '__main__':
    gradient_descent(1, 1, 0.001, 100)
```


Output:

```bash
1 (1.000000, 1.000000)
2 (-0.111111, 0.444444)
3 (0.074074, 0.074074)
4 (-0.008230, 0.032922)
5 (0.005487, 0.005487)
6 (-0.000610, 0.002439)
7 (0.000406, 0.000406)
8 (-0.000045, 0.000181)
```

##### matlab

matlab版的有点问题。。

```matlab
clc;clear;
e = 1e-3;
x0 = [1 1];
max_iters = 100;

gradient_descent_algorithm(e, x0, max_iters);

function gradient_descent_algorithm(epsilon, x0, max_iters)
f = @(p) 4 * p(1) + 2 * p(2);
[x,y] = meshgrid(-2:0.1:2);
z = 2 * x.^2 + y.^2;
figure
surfc(x,y,z);
xlabel('x');
ylabel('y');
zlabel('z');
[zx,zy] = gradient(z);
d = -x0;
iters = 0;
while sqrt(d * d') > epsilon && iters < max_iters
    inat = find(x == x0(1) & y == x0(2));
    d = -[zx(inat) zy(inat)];
    disp(d);
    syms c;
    l = f(x0 + c*d) == 0;
    lambda = solve(l, c);
    x0 = x0 + lambda*d;
    iters = iters + 1;
end
end

% function draw_search()
%
% end
```

### 参考文献

1.[《最优化理论与算法 第二版》・陈宝林](https://baike.baidu.com/item/%E6%9C%80%E4%BC%98%E5%8C%96%E7%90%86%E8%AE%BA%E4%B8%8E%E7%AE%97%E6%B3%95/8459488)
2.[Gradient descent - wikipedia](https://en.wikipedia.org/wiki/Gradient_descent)