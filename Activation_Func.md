# Activation Function

## ReLu

### Simple ReLU

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



### LeakyReLU

```math
LeakyReLU(x) = 
\left\{\begin{array}{ll}
x & \text { if } x \geq 0 \\
\alpha x & \text { if } x<0
\end{array}\right.
```

* LeakyReLU在输入值为正时，输出与ReLU相同，但**在输入值为负时，输出一个小的线性值**

* 适用场景：针对“死亡”神经元，在负输入情况下，LeakyReLU仍然有一个小的梯度（$\alpha $），使其权重在训练过程中仍然可以更新



### ELU

$$
\text{ELU}(x) = 
\begin{cases} 
x & \text{if } x > 0 \\
\alpha (\exp(x) - 1) & \text{if } x \leq 0
\end{cases}
$$

* $ \alpha $ 往往默认为1
* 特点：
  * **避免神经元死区**：与 ReLU（Rectified Linear Unit）不同，ELU 在输入小于零时仍有输出，这可以避免 ReLU 中的神经元死区问题（即输入始终为负时，梯度为零，神经元无法更新）
  * **带有负值的平滑输出**：ELU 的负值部分是平滑的指数函数，这可以使模型在训练时更稳定，并加速收敛
  * **零均值特性**：ELU 在输入均匀分布时，输出均值更接近于零，这有助于降低偏置偏差，从而提高模型的性能