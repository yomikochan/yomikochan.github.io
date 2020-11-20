---
title: 牛顿法(Newton's method)
layout: post
date: '2018-08-04 16:21:39 +0800'
tags:
- C/C++
- Math
categories:
- algorithms
- mathematical
cover_index: "/assets/222044074.jpg"
math: true
comments: true
---

### 算法简介

感觉是老生常谈的算法了（其实咱就是来水一下> <）。但还是稍微说一下这个算法：对于形如 $ y = f(x) = 0 $ 这样的方程，初中的时候大部分的方程我们都可以使用求根公式解决，但实际上有许多方程我们很难求出它的根，于是只能用数值分析方法来逼近根值。基本原理:设 $ y=f(x) $ 在 $ x \in [a,b] $  上连续，如果有 $ f(a)f(b) < 0 $ ，则 $ f(x), x \in (a, b) $ 一定过零点（记不得叫啥定理了，好像叫费马引理。至于为什么，画个图就很容易明白了。），那么 $ f(x) $ 在点 $ x0, (a < x_0 < b)$ 处的切线方程也过零点，根据 $ f(x) $ 曲线的类型：递增、递减、凸或凹，我们会发现起点的选择对于切线随着点向根方向移动时的效率也是不同的。

比如：若 $ x \in [a, b], f(a) < 0, f(b) > 0, f''(x) < 0 $ 时，起点应从a点开始迭代效率会更高，即考察 $ f(a) $ 与 $ f''(x) $ 是否同号，如果同号，则a点为起点，否则b点为起点。

总之，根据 $ y = f(x) $ 在 $ x_0 $ 处求出切线方程有：

$$ f(x) - f(x_0) = f'(x_0)(x - x_0) $$

由于 $ f(x) = 0 $，所以有：

$$ x_1 = x_0 - \frac{f(x_0)}{f'(x_0)} $$

再在点 $ (x_1, f(x_1)) $ 作切线，可得根得近似值 $ x_2 $.如此反复进行，一般的，在点(x_{n-1}, f(x_{n-1}))作切线，最终可得到迭代方程：

$$ x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)} $$

算法原理参考：高等数学。

想要再详细了解的同学可以看看：[Newton's method](https://en.wikipedia.org/wiki/Newton%27s_method)

### 算法实现

这里我使用函数指针方式来保证接口使用的便利性，虽然可变长参数还没做处理，这一点有空再完善吧。。。

```cpp
#include <iostream>
typedef double(*equaton)(double x, ...);
double square(double x)
{
	return x * x;
}
double func1(double x, ...)
{
	return square(x)*x + square(x) - 1.;
}
double func2(double x, ...)
{
	return 3 * square(x) + 2 * square(x);
}
double newton_method(equaton func, equaton der_func, double a, double b, bool isbump, double λ) // isbump： 凹 is true; 凸 is false
{
	if (func(a)*func(b) >= 0)
		return 0.0;
	double x, pre_x;
	if (func(a) > 0 && isbump)
		x = a;
	else
		x = b;
	do
	{
		pre_x = x;
		x -= func(x) / der_func(x);
	} while (abs(func(x)) > λ && fabs(pre_x - x) > λ);
	return x;
}
int main()
{
	std::cout << "方程的根为：" << newton_method(func1, func2, 0., 1., true, 1e-6) << std::endl;
	return 0;
}
```

测试：

$ f(x) = x^3 + x^2 - 1 $

output:

```
方程的根为：0.754878
```

更新：增加了两个函数，一个是 $ f(x) $ 的二阶导函数，一个是判断原函数凹凸。

```cpp
#include <iostream>
typedef double(*equaton)(double x, ...);
double square(double x)
{
	return x * x;
}
double func1(double x, ...)
{
	return cos(x) - square(x)*x;
}
double func2(double x, ...)
{
	return -sin(x) - 3 * square(x);
}
double func3(double x, ...)
{
	return -cos(x) - 6 * x;
}
bool isbump(double a, double b)
{
	for (double i = a; i <= b; i += 1e-2)
		if (func3(i) < 0.)
			return false;
	return true;
}
double newton_method(equaton func, equaton der_func, double a, double b, bool isc, double λ) // isc： 凹 is true; 凸 is false
{
	if (func(a)*func(b) >= 0.)
		return 0.0;
	double x, pre_x;
	if (func(a) > 0 && isc)
		x = a;
	else
		x = b;
	do
	{
		pre_x = x;
		x -= func(x) / der_func(x);
	} while (abs(func(x)) > λ && fabs(pre_x - x) > λ);
	return x;
}
int main()
{
	std::cout << "方程的根为：" << newton_method(func1, func2, 0., 1., isbump(0., 1.), 1e-6) << std::endl;
	return 0;
}
```

测试：

$ f(x) = cos(x) - x^3 $

output:

```
方程的根为：0.865474
```

![图1](img/cosx-x^3.png)

关于牛顿法开方（或任意次方）算法，强烈推荐看看这个回答：https://www.guokr.com/question/461510/

简单提一下需要注意得地方：求 $ \sqrt{2} $ 时，考虑有 $ x^2 - a = 0 $，即 $ x = \pm \sqrt{a} $，于是 $ x $ 就是迭代公式中的 $ x_i $，$ a = 2 $ 就是我们要求的。回答中给出了求任意次方时的公式，直接套用即可。

开方代码实现：

```cpp
#include <iostream>
int main()
{
	double x, pre_x;
	x = 1;
	pre_x = 9999;
	while (fabs(pre_x - x) > 0)
	{
		pre_x = x;
		x = (x + (2 / x)) / 2;
	}
	std::cout << x << std::endl;
	return 0;
}
```

牛顿法求 $ x^n $ 算法实现（代码有误）：

```cpp
#include <iostream>
double newton_method_power(double a, double b)
{
	if (a == 0.)
		return 0.;
	else if (a == 1. || b == 0.)
		return 1.;
	else if (b == 1.)
		return a;
	else if (fabs(b) > 1.)
		return 0.; // 当指数|b| > 1 且为整数时,使用快速幂算法；为浮点数时，可以使用微分近似法或泰勒公式求解
	else
	{
		b = 1./b;
		double x, pre_x;
		x = 1.;
		pre_x = 9999.;
		while (fabs(pre_x - x) > 0.)
		{
			pre_x = x;
			x = ((b - 1.)*x / b) + (a / (b*newton_method_power(x, b - 1.)));
		}
		return x;
	}
}
int main()
{	
	std::cout << newton_method_power(8., 1.) << std::endl;
	return 0;
}
```
微分近似公式：

$$ f(x + \Delta x) \approx f(x) + f'(x) \cdot \Delta x $$

例如：

1.求 $ 2^{1.4} $

由 $ f(x) = a^x $，$ f(x + \Delta x) = a^{x + \Delta x} \approx a^{x} + a^{x} ln(a) \cdot \Delta x $
有 $ 2^{1 + 0.4} \approx 2 + 2ln(2) \cdot 0.4 \approx 2.554518... $
而原表达式 $ 2^{1.4} = 2.63902... $

2.求 $ 2^{9} $

利用公式有 $ 2^{10 - 1} = (2^{10})^{(1 - \frac{1}{10})} \approx 2^{10} + 2^{10} ln(2^{10}) \cdot 0.1 \approx 709.783... $
$ 2^9 = 512 $

### 参考

1.[Newton's method - wikipedia](https://en.wikipedia.org/wiki/Newton%27s_method)
2.[果壳网](https://www.guokr.com/question/461510/)
