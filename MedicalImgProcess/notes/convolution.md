# Convolution

## dilated convolution

> reference：
>
> * 感受野：https://zhuanlan.zhihu.com/p/28492837
> * 空洞卷积：https://www.zhihu.com/question/54149221



## depthwise separable conv

* 深度可分离卷积：轻量级网络
* 在深度可分离卷积中，每个输入通道都被单独地卷积处理，然后结果被叠加在一起。这种操作可以减少计算量和模型参数，同时保持良好的模型性能。此处，每个卷积层的输入通道数逐渐增加，因为每个卷积层的输出都被叠加到下一个卷积层的输入中。

> reference：https://zhuanlan.zhihu.com/p/92134485