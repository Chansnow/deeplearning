
# Batch Normalization

为能顺利地进行学习，神经网络需要各层有适当的广度

BN的思路是调整各层的激活函数值分布使其拥有适当的广度

#### BN的优点

- 可以增大学习率使学习快速进行
- 不那么依赖初始值
- 抑制过拟合

#### 做法

**减均值除方差：远离饱和区**

向神经网络中插入对数据分布进行正规化的BN层，进行使数据分布的均值为0、方差为1的正规化

$$
\mu_B\leftarrow\frac{1}{m}\sum_{i=1}^mx_i
$$

$$
\delta_B^2\leftarrow\frac{1}{m}\sum_{i=1}^m(x_i-\mu_B)^2
$$

$$
\hat{x_i}\leftarrow \frac{x_i-\mu_B}{\sqrt{\delta_B^2+\varepsilon}}
$$

> $\varepsilon$为微小值，防止出现除以0的情况

先对每个batch的m个输入数据求均值 $\mu_B$ 和方差 $\delta_B^2$ ，然后对输入数据进行均值为0、方差为1的正规化

但这个处理对于在-1~1之间的梯度变化不大的激活函数，效果会变差

对sigmod来说，它在-1~1之间几乎呈线性，BN变换之后没有达到非线性变换的目的

对ReLU来说，会有一半置0

所以要使用缩放因子γ和移位因子β来进行一些转换将分布从0移开。

**缩放加移位：避免线性区**

$$
y_i\leftarrow\gamma\hat{x_i}+\beta
$$


---

<font color="red">梯度饱和问题</font>

在训练神经网络的过程中会碰到梯度消失和梯度爆炸的问题。

例如以sigmod作为激活函数的网络，sigmod为S型函数，随着激活层输出不断靠近0或是1，它的导数值逐渐接近0。偏向0和1的数据分布会造成反向传播中梯度值不断变小，最后消失，即梯度消失。

**梯度消失：** 在神经网络进行反向传播时，随着每层网络的反向反馈，靠近输入层的网络参数梯度越来越小，最终没有变化。此时网络参数将不会更新，训练不再收敛，loss过早地不再下降，精确度也过早地不再提高。

**梯度爆炸：** 靠近输入层的网络层，计算的到的偏导数极其大，各层需要更新很大的权重

由此可见，如果激活值的分布有所偏向，则在表现力上会有很大问题。所以我们希望网络各层的激活值呈现出具有相同广度的分布。

为解决梯度消失问题，我们需要仔细考虑网络权重的初始值。Batch Nomalization的提出又为我们提供了一种新的解决办法。

《Batch Normalization: Acclerating Deep Network Training by Reducing Internal Covariate Shift》提出了通过减少内部协变量偏移来加速深度网络的训练。BN的思路是调整网络各层的激活值分布使其拥有适当的广度。

**内部协变量偏移：** 在训练神经网络的过程中，每一层的输入分布随着前一层的参数变化而变化。这就需要较低的学习速率和仔细的参数初始化来减缓训练速度，并且使得具有饱和非线性的模型的训练变得非常困难。这一现象称为内部协变量偏移。内部协变量偏移会使得上层网络需要不停调整来适应输入数据分布的变化，导致网络学习速度的降低，同时网络的训练过程容易陷入梯度饱和区，减缓网络收敛速度。

在实践中，饱和问题和产生的梯度消失问题通常通过使用非饱和激活函数（如ReLU）、仔细的初始化和较小的学习率。

如果能确保非线性输入的分布在网络训练中保持更稳定，则优化器就不太可能陷入饱和状态，训练就能加速。

BN能减少梯度对参数或其初始值的尺度依赖性，，学习率可以取得较大。

BN提出对网络每层输入进行增白来使网络收敛得更快。

BN算法对白化做了如下简化：

1. 单独对每个特征进行归一；
2. 使用mini_batch计算对应的均值和偏差

---

#### BN算法

$$
\mu_B\leftarrow\frac{1}{m}\sum_{i=1}^mx_i
$$

$$
\delta_B^2\leftarrow\frac{1}{m}\sum_{i=1}^m(x_i-\mu_B)^2
$$

$$
\hat{x_i}\leftarrow \frac{x_i-\mu_B}{\sqrt{\delta_B^2+\varepsilon}}
$$

> $\varepsilon$为微小值，防止出现除以0的情况

先对每个batch的m个输入数据求均值$\mu_B$和方差$\delta_B^2$，然后对输入数据进行均值为0、方差为1的正规化

但这个处理对于在-1~1之间的梯度变化不大的激活函数，效果会变差

对sigmod来说，它在-1~1之间几乎呈线性，BN变换之后没有达到非线性变换的目的

对ReLU来说，会有一半置0，所以要使用缩放因子γ和移位因子β来进行一些转换将分布从0移开。

即做过上述处理后，每一层网络的输入数据分布变得稳定，却使大部分值落入了函数的线性部分。

**缩放加移位：避免线性区**

$$
y_i\leftarrow\gamma\hat{x_i}+\beta\equiv BN_{\gamma,\beta}(x_i)
$$

---

#### BN优势

1. 可以选择比较大的初始学习率，对网络参数不那么敏感，简化调参过程
2. 起到了一定的正则化效果
3. 允许网络使用非线性激活函数，缓解梯度消失问题
4. 使网络每一层数据分布相对稳定，加速模型学习速度



