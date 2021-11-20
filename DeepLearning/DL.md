# Deep Learning Notes



## resources

- 课程主页：https://courses.d2l.ai/zh-v2
- 教材：https://zh-v2.d2l.ai/
- 课程论坛讨论：https://discuss.d2l.ai/c/16
- pytorch论坛：https://discuss.pytorch.org/



## 基础数学

#### 基础运算

- 点乘（`.dot()`），矩阵对应位置元素相乘；
- 矩阵与向量（`.mv()`），类似于`broadcast`机制；

#### 矩阵行和与列和

```python
X = torch.tensor([[1.0, 2.0, 3.0],[4.0, 5.0, 6.0]])
```

$$
\begin{bmatrix}
1.0 & 2.0 & 3.0 \\
4.0 & 5.0 & 6.0 \\
\end{bmatrix}
$$

对于：

```python
X.sum(0, keepdim=True)
X.sum(1, keepdim=True)
```

结果为：
$$
\begin{align}
	\begin{bmatrix}
		5.0 & 7.0 & 9.0
	\end{bmatrix} 
	\tag{列和}
\\
\\
	\begin{bmatrix}
		6.0 \\ 
		15.0 \\
	\end{bmatrix} 
	\tag{行和}
\end{align}
$$


#### 导数





## 神经网络


$$
\boldsymbol{X}=[\boldsymbol{x_1}, \boldsymbol{x_2}, \boldsymbol{\cdots}, \boldsymbol{x_n}]^T \\ 

\boldsymbol{y}=[y_1, y_2, \cdots, y_n]^T
$$

### 线性回归

##### 推导

价格公式：$y=w_1x_1 + w_2x_2 + w_3x_3 + b$

向量版本：$\begin{pmatrix}\boldsymbol{w}, \boldsymbol{x}\end{pmatrix} + b$ 

损失函数：$\mathscr{L}(y,\hat{y}) = \frac{1}{2}(y - \hat{y})^2$

训练损失：$\mathscr{L}(\boldsymbol{X},\boldsymbol{y},\boldsymbol{w},b) = \frac{1}{2n}\sum_{i=1}^{n}(y_i - (\boldsymbol{x_i}, \boldsymbol{w}) - b)^2=\frac{1}{2n} \Vert{\boldsymbol{y} - \boldsymbol{Xw} - b}\Vert$

通过最小化损失来学习参数：$\boldsymbol{w^*}, \boldsymbol{b^*} = \arg \min_{\boldsymbol{w}, b} (\mathscr{L}(\boldsymbol{X},\boldsymbol{y},\boldsymbol{w},b))$

##### 显示解

$\boldsymbol{X} \gets [\boldsymbol{X}, 1] \qquad \boldsymbol{w} \gets \begin{bmatrix}\boldsymbol{w} \\ b\end{bmatrix}$

则有：

$\mathscr{L}(\boldsymbol{X},\boldsymbol{y},\boldsymbol{w}) = \frac{1}{2n}\Vert{\boldsymbol{y} - \boldsymbol{Xw}}\Vert^2 \to \frac{\partial{\mathscr{L}}}{\partial{\boldsymbol{w}}}=\frac{1}{n}(\boldsymbol{y}-\boldsymbol{Xw})^T\boldsymbol{X}$

所以最优解满足：

$\frac{\partial{\mathscr{L}}}{\partial{\boldsymbol{w}}}=0 \to \boldsymbol{w^*}=(\boldsymbol{X}^T\boldsymbol{X})^{-1}\boldsymbol{X}\boldsymbol{y}$

##### 梯度下降

- 随机挑选一个初始值$\boldsymbol{w}_0$；

- 开始迭代使$\boldsymbol{w}_t = \boldsymbol{w}_{t-1} - \eta\frac{\partial{\mathscr{L}}}{\partial{\boldsymbol{w}_{t-1}}}$；

  ![gradient](gradient.png)

  - 学习率（步长的超参数）$\eta$

- 小批量随机梯度下降：$\frac{1}{b}\sum_{i\in{I_b}}{\mathscr{L}(\boldsymbol{x}_i,y_i,\boldsymbol{w})}$

  - $b$为样本大小（批量大小），随机采样来近似损失；

---



### 分类问题

- 多输出，输出i是预测为第i类的置信度；

  ![image-20211116112358781](classify_network.png)

- 对类别进行有效编码：$\boldsymbol{y}=\begin{bmatrix}y_1,y_2,\cdots,y_n  \end{bmatrix}^T$

  其中：$y_i=\begin{cases}1 \qquad if \  i=y \\ 0 \qquad otherwise\end{cases}$

- 最大值为预测：$\hat{y}=\mathop{\arg\max}_{i}o_i$
- 更置信的识别正确类（大余量），即：$o_y-o_i\geq\Delta(y,i)$

##### $softmax$回归

- 希望输出是个概率（是否匹配），即：

  $\boldsymbol{\hat{y}}=softmax(\boldsymbol{o}) \leftrightarrow \hat{y}_i=\frac{\exp(o_i)}{\sum_{k}{\exp(o_k)}}$

- 概率的区别作为损失部分；

##### 交叉熵

$H(\boldsymbol{p},\boldsymbol{q}=\sum_{i}-p_i\log(q_i))$

$\mathscr{L}(\boldsymbol{y},\boldsymbol{\hat{y}}=-\sum_{i}{y_i \log{\hat{y}_i}})$

则其梯度为：

$\frac{\partial{\mathscr{L}}}{\partial{o_i}}=softmax(\boldsymbol{o})_i-y_i$