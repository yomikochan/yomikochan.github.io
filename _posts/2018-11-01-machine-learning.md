---
title: 机器学习
layout: page
date: '2018-10-24 22:34:16 +0800'
tags:
- Machine Leraning
- Python
math: true
comments: true
---

### 前言

机器学习可以做许多有趣的事情，每个技术方向都有入门例子，但实际上只是照着入门例子抄，然后跑一遍可能对入门并没有帮助。于是我就个人入门经验，用自己的例子来做一个入门指导吧。

虽然是入门，但这里并不会指导进行环境搭建，所以，在看这篇文章之前，请先准备好机器学习的环境。

### 你将会了解到的

1. 什么是数据集
2. 如何训练模型
3. 模型的复用

### 什么是数据集

我个人觉得入门不太合适一上来就教如何数据处理，特征工程，应该先对做机器学习过程有一个大体了解，然后再深入细节，学习更多细节方面的东西。

首先我的例子是这样的：

预测方程 $ a_n = a_{n - 2} + a_{n - 4} + 1 $ 的第 $ n $ 项的值。（不要求答案完全正确，误差在 1e7 左右）

正常来说，这种问题不会用机器学习来做，明明可以写个递归跑，干嘛非要机器学习，误差还很大？嘛...你就当计算非常大的项 $ n $ 时，跑递归计算很慢，觉得不爽，只想知道值大概范围即可。

那么你可以继续往下看。。。

我们首先要有数据集让机器能学习，如何弄到数据集呢，呃，监督学习似乎都可以用表格来存储数据并作为数据集，对于上面这个问题，显然是回归任务，它是属于监督学习的。

而数据集一般也分为训练集和测试集，顾名思义，训练集用于训练，测试集用来测试训练好的模型的效果如何。

下面Python代码用来生成问题的训练集（python很慢）：

```py
import pandas as pd


def recursive(x):
    if x <= 2:
        return 1
    else:
        return recursive(x - 2) + recursive(x - 4) + 1


x = []
y = []
c = 1
for i in range(0, 66):
    x.append(i)
    if i % 2 != 0:
        c = recursive(i)
    y.append(c)

save = pd.DataFrame({'data': x, 'result': y})
save.to_csv('train.csv', index=False)
```

这里我选择0~65用来训练，66~86用来测试。你可以用两个文件中分别保存训练集和测试集，也可以把所有数据放在同一个文件，然后训练时只取一部分。我用了两个文件。

**一般比赛时，主办方会提供数据集，所以，大家其实不太重视如何生成数据集，而更多的关注的是数据清洗等问题。**

### 如何训练模型

前面也提到这是一个回归任务，那么至少要知道应该用 sklearn 提供的 linear_model 来做这个问题。

问题很单一，所以我们不需要数据预处理。。。

不过为什么用回归任务的线性模块算法，看看数据分布关系，就明白了：

```py
import pandas as pd
import seaborn as sns

train = pd.read_csv('train.csv')
sns.lmplot(x='data', y='result', data=train)
plt.show()
```

![img1](/img/Figure_1.png)


然后很简单的拟合，并保存模型：

```py
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.linear_model import Lasso, RidgeCV, ElasticNet, BayesianRidge, LinearRegression, LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.externals import joblib
from sklearn.preprocessing import PolynomialFeatures


train = pd.read_csv('train.csv')
train_X, train_y = train.iloc[:, :-1].values, train['result']

poly_reg = PolynomialFeatures(degree=9) # 2333
X = poly_reg.fit_transform(train_X)
clf = LinearRegression()
print(clf.fit(X, train_y)) # 拟合

# c = clf.coef_
# print('参数：', c)

plt.scatter(train_X, train_y)
plt.plot(X, clf.predict(X))
plt.xlabel('data')
plt.ylabel('result')
plt.show()

joblib.dump(clf, 'save_model.m') # 拟合后，随意取个名字保存即可
```

把大多数回归模型可以用的算法都试过以后，发现还是非线性回归转线性回归最合适，误差也小一些。然而评分看着很高，却并没有什么用（笑）

### 模型的复用

上面保存了训练好的模型，我们随时可以用它来做事:

```py
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.metrics import mean_squared_error, r2_score
from sklearn.linear_model import Lasso, RidgeCV, ElasticNet, BayesianRidge, LinearRegression, LogisticRegression
from sklearn.externals import joblib
from sklearn.preprocessing import PolynomialFeatures

test = pd.read_csv('test.csv')
test_X, test_y = test.iloc[:, :-1].values, test['result']

clf = joblib.load('save_model.m')
poly_reg = PolynomialFeatures(degree=9)
X = poly_reg.fit_transform(test_X)
pred = clf.predict(X) # 预测
print(pred)
print('均方误差：%.2f' % mean_squared_error(y_true=test_y, y_pred=pred))
print('评分：%.2f' % r2_score(y_true=test_y, y_pred=pred))
```
