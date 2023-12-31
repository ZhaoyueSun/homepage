---
title: "《Elements of Causal Inference》笔记【中文】- CH1 - Statistical and Causal Models"
date: 2023-08-08
---

# Chapter 1 - Statistical and Causal Models

## 1.1 Probablity Theory and Statistics

> 变量：是能取多个值的任意属性或符号。可以把变量看作一个问题，而变量的值就是问题的答案。比如在临床试验中，“受试者的年龄”是变量，关于“受试者的年龄多大”的答案就是变量的值。  
> 事件：为一个变量或者一个变量集合指定一个值（或一组值）称为一个事件。比如“X=1”，“Y=1且Y=3”，“抛硬币正面朝上”，“调查对象大于40岁”等都是事件。  
> 概率分布：变量X的概率分布是X的每一个可能取值的概率的集合。在本书中用$P_X$表示。  
> 概率密度函数/概率质量函数： 是描述一个随机变量的输出值，在某个确定的取值点附近的可能性的函数。本书中用$p_X$表示变量X概率密度函数/概率质量函数，用$p(x)$表示$p_X$在$X=x$处的取值，即随机变量X的值为x的概率。  
> 条件概率分布、联合概率分布的表示类似  
—— 部分定义选自《统计因果推理入门（翻译版）》Judea Pearl 等著，杨矫云等译


概率论和统计学习可以说互为反向问题。他们都以随机试验为研究对象。概率论是给定数学模型，推断随机试验的结果。统计学习则是给定随机试验的结果，推断其潜在数学结构的性质。统计学习**假设**我们有一堆观察到的数据$(x_1,y_1)...,(x_n,y_n)$ (其中$x_i$是输入，$y_i$是输出)，并且这些数据是根据随机变量X和Y的联合概率分布P(X,Y)依**独立同分布**(independent and identically distributed, i.i.d.)产生的。我们希望根据这些数据推断联合概率分布P(X,Y)的有用的性质，比如：
- 1）回归问题： $f(x) = E[Y|X=x]$
- 2）分类问题： $f(x) = argmax_{y\in\mathcal{Y}}P(Y = y|X = x)$
- 3）生成模型，学习联合概率密度：$p_{X,Y}$

但实践中，独立同分布的假设很容易被打破，比如分布漂移或者需要对系统进行干预时，这些将于因果学习息息相关。

## 1.2 Learning Theory

<!-- 1.2.1 统计学习的病态问题 (ill-posed problem) -->

> 损失函数度量模型一次预测的好坏，风险函数度量平均意义下模型预测的好坏。  
——《统计学习方法》 李航著

统计学习的目标是选择期望风险（expected risk）最小的模型。期望风险即理论上模型$f(X)$关于联合分布$P(X,Y)$的平均意义下的损失：

$R_{exp}(f) = E_P[L(Y,f(X)))] = \int_{\mathcal{X}\times\mathcal{Y}}L(y,f(x))p(x,y)dxdy$

$L(Y,f(X))$指损失函数。

但是很遗憾$R_{exp}(f)$是不能直接计算的，因为联合概率分布是未知的，即式子中的p(x,y)是不知道的。（注：在《统计学习方法》中提到，统计学习一方面根据期望风险最小化来学习模型需要用到联合分布，另一方面联合分布又是未知的，所以构成了一个病态问题。本文中的病态问题是从另一个角度来讲的，见下文。）

由于期望风险不能直接计算（和最小化），我们只能寻求某些归纳原理，比如转而最小化经验风险（empirical risk, 记为$R_{emp}$），即可观测到的样本集上的平均损失：

$R_{emp}(f) = 1/N \sum_{i=1}^NL(y_i,f(x_i))$

这也就是说，我们要对一个我们观察不到的东西（潜在的联合分布P(X,Y)），基于在它上面的一些操作(比如，采样)得到的**有限的**结果进行的观察, 来估计这个未知的东西的性质。本书作者认为这是一个病态的问题，因为对于任何在采样中未见的输入x，其条件期望是未定义的 (undefined)。本书对这个问题的解释涉及一些（我本人）较为陌生的机器学习理论概念，如模型的容量（capacity）和一致性(consistency)等等，感兴趣的可以参考原书和这篇文章([BERNHARD SCHO¨LKOPF, 2003](https://onlinelibrary.wiley.com/doi/pdf/10.1002/cplx.10094))。下面简单说下我的理解：

![Illustration of model choice](/homepage/docs/assets/elements_of_causal_inference_ch1/model_choice.png)

*Figure 1. Graph from [BERNHARD SCHO¨LKOPF, 2003](https://onlinelibrary.wiley.com/doi/pdf/10.1002/cplx.10094). *

每个学习过（统计）机器学习的同学应该都对类似上面这个的图很熟悉。对于同一批观察到的数据，可以选择以简单的直线来拟合数据，也可以通过复杂的n次曲线来拟合。但如图中n次曲线虽然比直线更完美地匹配了数据，但对任何不属于已知样本的输入都可能造成结果的巨大波动。我们一般称这种情况为过拟合，并且倾向于选择更为“简单”的拟合模型。那多“简单”的模型才正好合适呢？统计学习理论通过capacity等度量形式化了哪些模型更可取，在此先略过。重要的是，无论我们有多少数据，有限的数据始终无法完美地估计真实的潜在分布，但可能的模型选择却有很多（假设空间很大），我们必须对模型进行一些假设（也称为归纳偏置，或先验），比如选择多简单的模型，才能进行良好的学习。

这便是**统计学习的病态问题**，即**任意规模的有限数据始终无法涵盖潜在模型的所有信息**。而**因果学习的病态性**则多了一个层次：**即便是知道观测分布的全部知识，也不一定能确定其下潜在的因果模型**。

## 1.3 Causal Modeling and Learning










