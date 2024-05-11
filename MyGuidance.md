# MyGuidance：

## Transformer

* [The Illustrated Transformer – Jay Alammar – Visualizing machine learning one concept at a time. (jalammar.github.io)](https://jalammar.github.io/illustrated-transformer/)



## In-Context Learning

* [什么是In-Context Learning（上下文学习）？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/611217770)



## Activation Function

### ReLu

$$
f(x) = max(0,x)
$$

* 从表达式可以看到ReLU函数实际上把大于0的数据都保留，而小于0的数据直接清零。这种“清零”实际上能够保证神经元的稀疏性，而这种“稀疏性”的好处则是能够降低计算量，并且能够防止过拟合，能够更好地挖掘图像特征。

* 优点：

  1. 没有饱和区，不存在梯度消失问题。

  2. 没有复杂的指数运算，计算简单、效率提高。

  3. 实际收敛速度较快，比 Sigmoid/tanh 等激活函数快很多。

  4. 比 Sigmoid 更符合生物学神经激活机制。

* 缺点：
  * 当学习率过大的时候，可能造成大部分神经元清零死亡，即该神经元不会对任何数据有激活反应