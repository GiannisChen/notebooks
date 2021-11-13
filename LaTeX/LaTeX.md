# LaTeX笔记



## 公式



#### 数学格式

##### 行内公式

```text
$ f(x) = a + b $
```

$ f(x)=a+b $



**行间公式**

```
$$ f(X) = a + b $$
```

$$
f(x)=a+b
$$



**手动编号**

```text
$$ f(x) = a - b \tag{1.1} $$
```

$$
f(x)=a-b \tag{1.1}
$$



#### 简单运算

##### 基本运算

| 运算符 |  公式  |   效果   |
| :----: | :----: | :------: |
| +-*/=  | +-*/=  | $ +-*/ $ |
|  点乘  | \cdot  | $\cdot$  |
|  不等  |  \neq  |  $\neq$  |
|   除   |  \div  | $ \div $ |
|  mod   | \bmod  | $\bmod$  |
|  恒等  | \equiv | $\equiv$ |
|  求导  |   ''   | $x' y''$ |



##### 根号、分式

| 运算符  |      公式       |
| :-----: | :-------------: |
| 平方根  |   \sqrt{...}    |
| n次方根 |  \sqrt[n]{...}  |
|  分式   | \frac{...}{...} |

$$
\sqrt{x^3}=\sqrt[4]{x^4+\frac{ac}{b^2}}
$$



##### 上下标记（共轭，无穷小数）

| 运算符 |            公式            |
| :----: | :------------------------: |
| 水平线 | \overline{}  \underline{}  |
| 大括号 | \overbrace{} \underbrace{} |
|   帽   |            \hat            |
|   拔   |            \bar            |
| 波浪号 |           \tilde           |

$$
\overline{x+y} \qquad \underline{x+y} \qquad \overbrace{1+2+..+n}^{n} \qquad \underbrace{1+2+...+n}_{n}
$$

$$
\hat{x} \qquad \bar{x} \qquad \tilde{x}
$$



##### 向量

| 运算符 |                公式                |
| :----: | :--------------------------------: |
|  基础  |               \vec{}               |
| 有方向 | \overrightarrow{} \overleftarrow{} |

$$
\vec{x} \qquad \overrightarrow{xy} \qquad \overleftarrow{xy} 
$$



##### 积分、极限、求和、求积

| 运算符 |            公式            |
| :----: | :------------------------: |
|  积分  | \int_{}^{} ... \mathrm{d}x |
|  极限  |   \lim{x \to \infty} ...   |
|  求和  |       \sum_{}^{} ...       |
|  求积  |      \prod_{}^{} ...       |

$$
\lim_{x \to \infty} \frac{1}{x} \qquad \int_{-\infty}^{+\infty}x\mathrm{d}x
$$

$$
\sum_{i=1}^{n} i^{2} \qquad \prod_{i=1}^{n} i^2
$$



##### 三圆点

|  运算符  |  公式  |
| :------: | :----: |
| 水平基线 | \ldots |
| 水平居中 | \cdots |
|   垂直   | \vdots |
|  对角线  | \ddots |

$$
\ldots \qquad \cdots \qquad \vdots \qquad \ddots
$$



##### 矩阵

| 运算符  |                     公式                      |
| :-----: | :-------------------------------------------: |
| matrix  |  $\begin{matrix} a & b \\ c& d \end{matrix}$  |
| bmatrix | $\begin{bmatrix} a & b \\ c& d \end{bmatrix}$ |
| vmatrix | $\begin{vmatrix} a & b \\ c& d \end{vmatrix}$ |
| pmatrix | $\begin{pmatrix} a & b \\ c& d \end{pmatrix}$ |
| 行分割  |                       &                       |
| 列分割  |                      \\                       |



##### 希腊字母

![preview](H:\notebooks\DeepLearning\greek.jpg)



#### 多行公式

##### 公式组合

```text
D(x) = \begin{cases}
\lim\limits_{x \to 0} \frac{a^x}{b+c}, & x<3 \\
\pi, & x=3 \\
\int_a^{3b}x_{ij}+e^2 \mathrm{d}x,& x>3 \\
\end{cases}
```

$$
D(x) = \begin{cases}
\alpha, & x<1 \\
\beta, & 1\leq x \leq 2 \\
\gamma, & x>2
\end{cases}
$$



##### 公式拆分

```text
\begin{split}
\cos 2x &= \cos^2x - \sin^2x \\
&=2\cos^2x-1
\end{aplit}
```

$$
\begin{split}
\cos 2x &= \cos^2x - \sin^2x \\
&=2\cos^2x-1
\end{split}
$$

