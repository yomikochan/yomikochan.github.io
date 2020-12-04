---
title: 梯度下降算法（更新于 2020/12/04）
layout: post
date: '2018-07-26 23:30:49 +0800'
tags:
- Math
- C/C++
- MatLab
- Python
categories:
- algorithms
cover_index: "/assets/60572969.jpg"
math: true
comments: true
---

[梯度下降法](https://en.wikipedia.org/wiki/Gradient_descent)(Gradient descent)又叫最速下降法，是一种局部搜索算法，属于智能算法，类似的有爬山算法、模拟退火等。基本原理是优化理论中的无约束问题，利用梯度搜索函数变化最快的方向，然后不断迭代寻找出极值的方法。

有空我将补上推导过程并给出梯度法的收敛条件，可能还会介绍一些其它无约束问题的优化算法(牛顿法、共轭梯度法等)及其他约束问题的优化算法，届时我们将会看到许多有趣的搜索算法，比如斐波那契法、黄金比例法(0.618法)等。

# 代码实现
使用方法就是自定义一个多元函数 $$ g $$，然后调用 `(gradient-descent g init-arguments max-iterations tolerance)` (e.g. $$ f(x_1, x_2) = exp(x_1^2 + x_2^2) $$，`(gradient-descent (lambda (vec) (exp (+ (square (list-ref vec 0)) (square (list-ref vec 1))))) (list 1.0 1.0) 30 0.00001)`)

Lisp:
```scheme
(define delta 0.00001)

(define (deriv g)
  (lambda (x)
    (/ (- (g (+ x delta)) (g x))
       delta)))

(define (newton-transform g)
  (lambda (x)
    (- x (/ (g x) ((deriv g) x)))))

(define (fixed-point g first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) delta))
  (define (try guess)
    (let ((next (g guess)))
      (if (close-enough? guess next)
	  guess
	  (try next))))
  (try first-guess))

(define (newton-method g guess)
  (fixed-point (newton-transform g) guess))

(define (minimized g)
  (newton-method (deriv g) 1.0))

(define (gradient-descent g init-vec max-it tolerance)
  (define len (- (length init-vec) 1))
  (define (partial-derivative g)
    (lambda (vec index)
      (let ((d-vec (list-copy vec)))
	(list-set! d-vec index (+ (list-ref d-vec index) delta))
	(/ (- (g d-vec) (g vec)) delta))))
  (define (gradient-vector g vec)
    (define (iter i res)
      (if (< i 0)
	  res
	  (iter (- i 1) (cons ((partial-derivative g) vec i) res))))
    (iter len ()))
  (define (distance vec)
    (define (iter i result)
      (if (< i 0)
	  result
	  (iter (- i 1) (+ result (square (list-ref vec i))))))
    (sqrt (iter len 0)))
  (define (next vec x grad)
    (define (iter i res)
      (if (< i 0)
	  res
	  (iter (- i 1) (cons (- (list-ref vec i) (* x (list-ref grad i))) res))))
    (iter len ()))
  (define (x-transform vec x grad)
    (next vec x grad))
  (define (iter vec i)
    (let ((grad-vec (gradient-vector g vec)))
      (display vec)
      (newline)
      (cond ((< max-it i) vec)
	    ((< (distance grad-vec) tolerance) vec)
	    (else (let ((minimum (minimized
				  (lambda (x)
				    (g (x-transform vec x grad-vec))))))
		    (iter (next vec minimum grad-vec) (+ i 1)))))))
  (iter init-vec 0))
```

# 测试
以 $$ min \ f(\mathbf{x}) = 2x_1 ^2 + x_2 ^2, $$ 起始点 $$ x_0 = (1, 1)^T, \epsilon = \frac{1}{100000} $$ 为例。

函数的样子：

![f(x) = 2x_1 ^2 + x_2 ^2](/img/funcgraph.png)

```scheme
(gradient-descent (lambda (vec) (+ (* 2 (square (list-ref vec 0))) (square (list-ref vec 1)))) (list 1.0 1.0) 50 0.00001)
```

输出：

```
(1. 1.)
(4.00002000002786 2.00001000001393)
(-.11109809785453906 .44445095107273047)
(-.44437239141992263 .8889119021471269)
(.07406415148073814 .0740568166924851)
(.2962766059225186 .14812363338459478)
(-8.226060108502659e-3 3.2915785342297756e-2)
(-.03288424043400382 .06584157068460685)
(5.479613029965878e-3 5.473976292833832e-3)
(2.1938452119861993e-2 .01095795258566738)
(-6.076458754474259e-4 2.4334748892817517e-3)
(-2.4105835017897382e-3 4.876949778563498e-3)
(3.998023673891758e-4 3.952652685951883e-4)
(1.6192094695567005e-3 8.005305371903731e-4)
(-4.351939188329914e-5 1.760888045421857e-4)
(-1.540775675331962e-4 3.621776090843713e-4)
(2.3048444806604486e-5 1.9613203841429633e-5)
(1.1219377922641792e-4 4.9226407682859245e-5)
(-1.9005725715702401e-6 8.666514023037452e-6)
(1.2397709713719043e-5 2.7333028046074907e-5)
(-4.13145223743898e-6 3.7481300728847094e-6)
(3.4741910502440802e-6 1.749626014576942e-5)
(-4.519310966043615e-6 1.7948473900002134e-6)
(1.9227561358255414e-6 1.358969478000043e-5)
(-4.58949997889981e-6 1.2987640626074323e-6)
(1.6420000844007603e-6 1.2597528125214866e-5)
(-4.602395677437769e-6 1.1998274507463487e-6)
(1.590417290248925e-6 1.23996549014927e-5)
(-4.604772161673363e-6 1.1812992414638758e-6)
(1.58091135330655e-6 1.2362598482927753e-5)
(-4.605206860686667e-6 1.1778999304581313e-6)
(1.5791725572533344e-6 1.2355799860916266e-5)
(-4.605288702309115e-6 1.1772595832695892e-6)
(1.5788451907635412e-6 1.2354519166539175e-5)
(-4.605301020582233e-6 1.1771631923490466e-6)
(1.5787959176710698e-6 1.2354326384698093e-5)
(-4.6053101069904185e-6 1.1770920897757949e-6)
(1.5787595720383275e-6 1.2354184179551592e-5)
(-4.605310439480507e-6 1.1770894879586069e-6)
(1.5787582420779763e-6 1.2354178975917217e-5)
(-4.6053066942518565e-6 1.177118795311575e-6)
(1.5787732229925766e-6 1.2354237590623149e-5)
(-4.605302200151839e-6 1.1771539626038494e-6)
(1.5787911993926453e-6 1.2354307925207699e-5)
(-4.60530742411387e-6 1.1771130842181956e-6)
(1.5787703035445227e-6 1.2354226168436392e-5)
(-4.6053079358607e-6 1.1771090796862664e-6)
(1.578768256557201e-6 1.2354218159372532e-5)
(-4.605308847380179e-6 1.177101946840487e-6)
(1.5787646104792857e-6 1.2354203893680974e-5)
(-4.605311482350731e-6 1.1770813275770237e-6)
(1.5787540705970797e-6 1.2354162655154049e-5)
(-4.60530989537697e-6 1.1770937460609003e-6)
(1.578760418492121e-6 1.2354187492121804e-5)
(-4.605310821719313e-6 1.1770864972048648e-6)
(1.5787567131227508e-6 1.235417299440973e-5)
(-4.605305752686176e-6 1.17712616367898e-6)
(1.5787769892552975e-6 1.235425232735796e-5)
(-4.6053072034125116e-6 1.1771148114491652e-6)
(1.5787711863499554e-6 1.2354229622898331e-5)
(-4.6053095736310096e-6 1.1770962639713363e-6)
(1.5787617054759638e-6 1.2354192527942674e-5)
(-4.6053081399419605e-6 1.1771074829352073e-6)
(1.5787674402321597e-6 1.2354214965870419e-5)
(-4.605309727121988e-6 1.1770950628898276e-6)
(1.5787610915120497e-6 1.2354190125779656e-5)
(-4.605306212339686e-6 1.1771225669175347e-6)
(1.5787751506412563e-6 1.2354245133835069e-5)
(-4.605307890314314e-6 1.1771094364157867e-6)
(1.578768438742746e-6 1.2354218872831574e-5)
(-4.605309945571984e-6 1.1770933535612826e-6)
(1.578760217712065e-6 1.2354186707122569e-5)
(-4.6053032700932465e-6 1.1771455908230034e-6)
(1.5787869196270146e-6 1.2354291181646009e-5)
(-4.6053068578616514e-6 1.1771175158909061e-6)
(1.5787725685533955e-6 1.2354235031781813e-5)
(-4.605308083785578e-6 1.1771079227724185e-6)
(1.5787676648576918e-6 1.2354215845544836e-5)
(-4.605314663462227e-6 1.1770564353070187e-6)
(1.5787413461510942e-6 1.2354112870614038e-5)
(-4.605306778623826e-6 1.1771181364740988e-6)
(1.5787728855046987e-6 1.2354236272948201e-5)
(-4.605302563391973e-6 1.1771511215669718e-6)
(1.5787897464321104e-6 1.2354302243133943e-5)
(-4.605305105647433e-6 1.1771312279792406e-6)
(1.5787795774102697e-6 1.2354262455958482e-5)
(-4.605305708265893e-6 1.177126512370606e-6)
(1.5787771669364311e-6 1.2354253024741213e-5)
(-4.605304422228972e-6 1.1771365758715509e-6)
(1.5787823110841116e-6 1.2354273151743104e-5)
(-4.605306645935686e-6 1.1771191749417624e-6)
(1.5787734162572566e-6 1.2354238349883525e-5)
(-4.60530458062508e-6 1.1771353364372822e-6)
(1.5787816774996827e-6 1.2354270672874563e-5)
(-4.6053073287780956e-6 1.177113831610685e-6)
(1.5787706848876187e-6 1.2354227663221368e-5)
(-4.605304935080639e-6 1.1771325628194816e-6)
(1.5787802596774463e-6 1.2354265125638964e-5)
(-4.605304972897239e-6 1.1771322668971706e-6)
(1.5787801084110472e-6 1.235426453379434e-5)
(-4.605309019871579e-6 1.1771005985278468e-6)
(1.5787639205136867e-6 1.2354201197055695e-5)
(-4.605308341516989e-6 1.177105906813003e-6)
(1.5787666339320441e-6 1.235421181362601e-5)
;Value: (-4.605308341516989e-6 1.177105906813003e-6)

```
问题最优解：`(0, 0)`

C++（有个问题，原因是 cpp 的lambda的对变量的捕获问题或者作用域的问题，如果不包装到class里面，每个功能定义为各个函数组件，那么只需要把向量作为全局变量，就可以正常运行，但如果非要用 class 包装的话，我暂时没找到解决办法）:
```cpp
#include <cmath>
#include <iostream>
#include <vector>
#include <cassert>
#include <functional>

const double h = 1e-5;

auto square(double x) -> double const { return x * x; }
auto cube(double x) -> double const { return x * x * x; }

template<class Fn_Ty>
auto partial_derivative(Fn_Ty g) {
    auto a = [&](const auto& vec, size_t i)
    {
        auto components = vec;
        components[i] += h;
        return (g(components) - g(vec)) / h;
    };
    return a;
}

template<class Fn_Ty, class Arg>
auto fixed_point(Fn_Ty g, Arg first_guess)
{
    auto close_enough = [](auto v1, auto v2)
    {
        return (std::abs(v1 - v2) < 1e-5);
    };
    auto try_again = [&](auto guess, auto& try_ref)
    {
        auto next = g(guess);
        if (close_enough(guess, next)) return next;
        else return try_ref(next, try_ref);
    };
    return try_again(first_guess, try_again);
}

template<class Fn_Ty>
auto deriv(Fn_Ty g)
{
    auto a = [&](auto x)
    {
        return (g(x + h) - g(x)) / h;
    };
    return a;
}

template<class Fn_Ty>
auto newton_transform(Fn_Ty g)
{
    auto a = [&](auto x)
    {
        return x - (g(x) / deriv(g)(x));
    };
    return a;
}

template<class Fn_Ty, class Arg>
auto newton_method(Fn_Ty g, Arg guess)
{
    return fixed_point(newton_transform(g), guess);
}

std::vector<double> operator*(double b, const std::vector<double>& a)
{
    std::vector<double> c(a.size());
    for (size_t i = 0; i < a.size(); i++) c[i] = a[i] * b;
    return c;
}

std::vector<double> operator*(const std::vector<double>& a, double b)
{
    return b * a;
}

std::vector<double> operator-(const std::vector<double>& a, double b)
{
    std::vector<double> c(a.size());
    for (size_t i = 0; i < a.size(); i++) c[i] = a[i] - b;
    return c;
}

std::vector<double> operator-(double b, const std::vector<double>& a)
{
    std::vector<double> c(a.size());
    for (size_t i = 0; i < a.size(); i++) c[i] = b - a[i];
    return c;
}

std::vector<double> operator-(const std::vector<double>& a)
{
    std::vector<double> c(a.size());
    for (size_t i = 0; i < a.size(); i++) c[i] = -a[i];
    return c;
}

void operator+=(std::vector<double>& a, const std::vector<double>& b)
{
    assert(a.size() == b.size());
    for (size_t i = 0; i < a.size(); i++) a[i] += b[i];
}

void operator-=(std::vector<double>& a, const std::vector<double>& b)
{
    assert(a.size() == b.size());
    for (size_t i = 0; i < a.size(); i++) a[i] -= b[i];
}

template<class Goal_FnTy>
class GradientDescent {
public:
    GradientDescent(std::function<Goal_FnTy> g, std::vector<double> init_values, const double error, size_t max_iter = 50) :
        error(error), max_it(max_iter), gradient_components(), init_variable(init_values), vec(init_values), goal_fn(g)
    {
    }

    void run(bool output = true)
    {
        size_t step = 0;
        while (step < max_it)
        {
            if (output) step_output(step);

            for (size_t i = 0; i < vec.size(); i++)
                gradient_components.push_back(partial_derivative(goal_fn)(vec, i));

            if (distance(gradient_components) < error) break;

            auto fg = [&](auto lambda)
            {
                std::vector<double> lambda_components;
                for (size_t i = 0; i < vec.size(); i++)
                    lambda_components.push_back(vec[i] - lambda * gradient_components[i]);
                return goal_fn(lambda_components);
            };

            // 使用牛顿法搜索极小点
            double min_lambda = minimization(fg);  // 极小点
            vec += min_lambda * (-gradient_components); // 下一步迭代
            gradient_components.clear();
        }
    }

    std::vector<double>& const run() const
    {
        run(false);
        return vec;
    }

    std::vector<double>& const get() const
    {
        return vec;
    }
    
private:
    void step_output(size_t step)
    {
        std::cout << step << " : " << "(";
        for (size_t i = 0; i < vec.size(); i++)
            std::cout << vec[i] << (i + 1 == vec.size() ? "" : ", ");
        std::cout << ")" << std::endl;
    }

    double distance(const std::vector<double>& vec)
    {
        double sum = 0;
        for (auto _ : vec) sum += square(_);
        return std::sqrt(sum);
    }

    template<class Fn_Ty>
    auto minimization(Fn_Ty g)
    {
        return newton_method(deriv(g), 1.0);
    }

    // 误差
    const double error;
    // 最大迭代次数
    size_t max_it;
    // 梯度向量
    std::vector<double> gradient_components;
    // 初始值
    std::vector<double> init_variable;
    // 最终目标函数极值
    std::vector<double> vec;
    // 目标函数
    std::function<Goal_FnTy> goal_fn;
};

int main()
{
    //std::cout << newton_method(deriv([](double x) { return 2. * square(1. - 4. * x) + square(1. - 2. * x); }), 1.0);
    auto my_func = [](const auto& x)
    {
        return square(x[0] + x[1]);
    };
    
    GradientDescent<double(const std::vector<double>&)> gd = GradientDescent<double(const std::vector<double>&)>(my_func, {1.0, 1.0}, 1e-5);
    gd.run();
    return 0;
}
```

以下部分是以前写的，暂时没更新。
<hr>

python:
```py
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
```
1 (1.000000, 1.000000)
2 (-0.111111, 0.444444)
3 (0.074074, 0.074074)
4 (-0.008230, 0.032922)
5 (0.005487, 0.005487)
6 (-0.000610, 0.002439)
7 (0.000406, 0.000406)
8 (-0.000045, 0.000181)
```

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

# 参考文献
1. [最优化理论与算法 第二版・陈宝林](https://baike.baidu.com/item/%E6%9C%80%E4%BC%98%E5%8C%96%E7%90%86%E8%AE%BA%E4%B8%8E%E7%AE%97%E6%B3%95/8459488)
2. [Gradient descent - wikipedia](https://en.wikipedia.org/wiki/Gradient_descent)
