## 排列组合（Permutation and Combination）

#### 加法原理

- 完成一个工程可以有 $n$ 类办法， $a_i(1 \leq i \leq n)$ 代表第 $n$ 类方法的数目。那么完成这件事共有 $S=a_1+a_2+ \cdots + a_n$ 种不同的方法。

#### 乘法原理

- 完成一个工程需要分 $n$ 个步骤，$a_i(1 \leq i \leq n)$ 代表第 $i$ 个步骤的不同方法数目。那么完成这件事共有 $S=a_1 \times a_2 \times \cdots \times a_n$ 种不同的方法。

#### 排列数

- 从 $n$ 个不同元素中，任取 $m$ （ $m \leq n$ ， $m$ 与 $n$ 均为自然数，下同）个元素按照一定的顺序**排成一列**，叫做从 $n$ 个不同元素中取出 $m$ 个元素的一个排列，所有排列的个数，叫排列数，用符号 $A_n^m$ （或者是 $P_n^m$ ）表示。**排列的计算公式**如下：
  $$
  A_n^m = n (n-1) (n-2) \cdots (n-m+1)= \frac{n!}{(n-m)!}
  $$

- **环排列**，在 $n$ 个元素中取出 $m$ 个元素排成一个环（考虑**位移相似**），其计算公式如下：
  $$
  Q_n^m = \frac{A_n^m}{m} = \frac{n!}{m \cdot (n-m)!}
  $$
  进一步考虑**顺时针**和**逆时针**无差别，那么公式如下：
  $$
  {Q'}_n^m = \frac{A_n^m}{2m} = \frac{n!}{2m \cdot (n-m)!}
  $$

- **全排列**：
  $$
  A_n^m = n!
  $$

#### 组合数

- 从 $n$ 个不同元素中，任取 $m$ （ $m \leq n$ ， $m$ 与 $n$ 均为自然数，下同）个元素组成一个集合，叫做从 $n$ 个不同元素中取出 $m$ 个元素的一个组合，所有组合的个数，叫组合数，用符号 $C_n^m$ 表示。**组合的计算公式**如下：
  $$
  C_n^m = \frac{A_n^m}{m!} = \frac{n!}{m!(n-m)!} = {n \choose m}
  $$

#### 插板法

- **插板法**（`Stars and bars`）是用于求一类给相同元素分组的方案数的一种技巧，也可以用于求一类线性不定方程的解的组数。

> **问题一**：现有 $n$ 个完全相同的元素，要求将其分为 $k$ 组，保证每组至少有一个元素，一共有多少种分法？

- 考虑拿 $k-1$ 块板子插入到 $n$ 个元素两两形成的 $n-1$ 个空里面。因为元素是完全相同的，所以答案就是 ：
  $$
  {n-1 \choose k-1}
  $$

- 本质上是求 $x_1 + x_2 + \cdots + x_k = n$ 的正整数解的组数。

> **问题二**：现有 $n$ 个完全相同的元素，要求将其分为 $k$ 组，每组允许为空，一共有多少种分法？

- 显然此时没法直接插板了，因为有可能出现很多块板子插到一个空里面的情况。我们考虑创造条件转化成有限制的问题一，先借 $k$ 个元素过来，在这 $n+k$ 个元素形成的 $n+k-1$ 个空里面插板，答案为：
  $$
  {n+k-1 \choose k-1} = {n+k-1 \choose n}
  $$

- 本质上是求 $x_1 + x_2 + \cdots + x_k = n$ 的非负整数解的组数。

> **问题三**：现有 $n$ 个完全相同的元素，要求将其分为 $k$ 组，第 $i$ 组至少分到 $a_i$ 个，一共有多少种分法？

- 类比问题二，先借 $\sum a_i$ 个元素过来，也就是令 $x_i' = x_i - a_i$ ，那么：
  $$
  \begin{align}
  (x_1' + a_1)+(x_2' + a_2) + \cdots + (x_k' + a_k) &= n \\
  x_1' + x_2' + \cdots + x_k' &= n - a_1 - a_2 - \cdots - a_k \\
  x_1' + x_2' + \cdots + x_k' &= n- \sum a_i
  \end{align}
  $$
  所以，用插板法得到答案：
  $$
  {n + \sum a_i -1 \choose n}
  $$

#### 不相邻的排列

> **问题**：现有 $n$ 个连续的自然数，要求选择其中 $k$ 个，保证两两都不想邻，一共有多少种选法？

$$
{n-k+1 \choose k}
$$

#### 二项式定理

- 从常见的公式：
  $$
  (a+b)^n = \sum_{i=0}^{n} {n \choose i} a^{n-i}b^{i}
  $$
  当等式两边同乘 $(a+b)$ ，则可以得到：
  $$
  {n \choose k} + {n \choose k-1} = {n+1 \choose k}
  $$

#### 多重集

- **多重集**是指包含重复元素的广义集合，即 $S = \{n_1 \cdot a_1, n_2 \cdot a_2, \cdots, n_k \cdot a_k\}$ ，为有 $n_1$ 个 $a_1$ ， $n_2$ 个 $a_2$ ， $\cdots$， $n_k$ 个 $a_k$ 。

##### 多重集的排列数

- **多重集的排列数**相当于把相同元素的排列数除掉了。具体地，你可以认为你有 $k$ 种不一样的球，每种球的个数为 $n_1 \sim n_k$ ，且 $n_1 + n_2 + \cdots + n_k = n$ 。这 $n$ 个球的全排列数就是**多重集的排列数**：
  $$
  {n \choose {n_1,n_2,\cdots,c_k}} = \frac{n!}{\prod_{i=1}^{k}n_i!}
  $$

##### 多重集的组合数

> **问题**：要求从多重集中取 $r$ 个元素组成新的多重集，一共有多少种取法？

- 显然转换成 $x_1 + x_2 + \cdots + x_k = r$：

  1. 如果有限制 $r \leq \min n_i$ ，那么很简单只要用插板法：
     $$
     {r+k-1 \choose k-1}
     $$

  2. 如果不加限制，则（我懒得自己推一遍，直接给答案）：
     $$
     \sum_{p=0}^{k}(-1)^p\sum_A {k+r-1 -\sum_AnA_i -p \choose k-1}
     $$
     其中 $A$ 是充当枚举子集的作用，满足 $|A| = p, A_i < A_{i+1}$ 。

#### 其他

- **见**：https://oi-wiki.org/math/combinatorics/combination/#%E7%BB%84%E5%90%88%E6%95%B0%E6%80%A7%E8%B4%A8--%E4%BA%8C%E9%A1%B9%E5%BC%8F%E6%8E%A8%E8%AE%BA



#### 例题

- 思路来源：https://leetcode.cn/problems/count-the-number-of-ideal-arrays/solution/shu-lun-zu-he-shu-xue-zuo-fa-by-endlessc-iouh/；
- 以题 [2338. Count the Number of Ideal Arrays](https://leetcode.cn/problems/count-the-number-of-ideal-arrays/) 为例，讲一下组合数学的妙用（`Rating 2615` ，高低给他磕一个...）

##### 朴素的开始

> - You are given two integers `n` and `maxValue`, which are used to describe an **ideal array**.
>
>
> - A **0-indexed** integer array `arr` of length `n` is considered **ideal** if the following conditions hold:
>
>   - Every `arr[i]` is a value from `1` to `maxValue`, for `0 <= i < n`.
>
>   - Every `arr[i]` is **divisible** by `arr[i - 1]`, for `0 < i < n`.
>
> - Return the number of distinct **ideal arrays** of length `n`. Since the answer may be very large, return it modulo `109 + 7`.

1. 首先看到这道题目，上来就是一个 `DP` ，然后不出意外就超时了。`DP` 的思路很直白，我确定了每一个 **ideal array** 的最后一个元素 `arr[maxValue]` ，那么，其子数组 `arr[1:maxValue]` 也是一个 **ideal array**，并且其末尾元素能整除 `arr[maxValue]`，这就有递推方程：
   $$
   dp[i] = dp[i] + \sum_{j=1}^{i}dp[j] \qquad\qquad j \; mod \; i 
   $$
   会超时。但是中间一个思路是可以延续下来的，那就是固定了最后一个元素，其向前递归的过程就是可控的。

2. 自然而然会想到，既然遍历耗时，那我能不能找规律用**数学方法**呢？先找规律，对于 `arr` 结尾为 `4` ，我们可以写出如下数组：（不妨假设 `n == 4`）

   ```python
   [1,1,1,4]	[2,2,4,4]
   [1,1,2,4]	[2,4,4,4]
   [1,2,2,4]	[4,4,4,4]
   [2,2,2,4]	...
   ```

   有趣的是，这是一个单调数组，而且很像前缀和的表达： `arr[i] = arr[i-1] * j` 。那么，我们可以用类似差分数组的方式搞一个乘法版的差分数组，`[1,2,2,4]` 👉 `[1,2,1,2]`，显然，包括了 `1` 作为初始化值，所有数字的乘积必然是 `4` ，换做其他数字也是一样的结果。那么，我们就只需要考虑数字 `4` 的因子分散到哪个地方了。

3. 不失一般性，有 `n` 个空盒子可以填入相应的因子，可以是多个，也可以不填入。自然而然想到了经典的**空盒子填球的问题**，但是还有一个问题，数字有多个因子如何处理。比如数字 `12` ，能被拆分成： `[12,6,4,3,2,1]` ，不太好考虑；差分数组中以 `1` 为基本单位，那么，我们自然想到**整除中**的最小单位：**质因子**。那么， `12` 就拆成了： `[2,2,3]`。

4. 这似乎是一个**多重集**的问题，但是我们发现，质因子之间由于**互质**的特性，选取并不影响彼此，只有相同因子之间需要考虑重复的影响。所以，`k` 个相同的质因子，放入 `n` 个空盒子，允许为空，**插板法**！先借 `n` 个数字，考虑至少放置 `1` 个，即 `k+n-1` 个空格中插入 `n-1` 块板子，结果就是 ${k+n-1 \choose n-1} = {k+n-1 \choose k}$。

5. 就差一步了，我们只要计算上述那个公式求和就行了，那么这个公式如何求呢，考虑 $n \choose k$ ，能不能用递推呢，答案是可以的，考虑到第 `n` 位，有两种情况，在此处选择 `k` 中的一个，那么剩下的情况总数就是 $n-1 \choose k-1$，同理，如果不选，就是 $n-1 \choose k$，所以有以下公式：
   $$
   {n \choose k} = {n-1 \choose k-1} + {n-1 \choose k}
   $$

6. 动手写下第一版的代码：

   ```go
   func idealArrays(n int, maxValue int) int {
   	ans := 0
   	for i := 1; i <= maxValue; i++ {
   		j := 2
   		tmp := i
   		a := 1
   		for tmp > 1 {
   			count := 0
   			for tmp%j == 0 {
   				count++
   				tmp /= j
   			}
   			if count > 0 {
   				a = (a * choose(count+n-1, count)) % (1e9 + 7)
   			}
   			j++
   		}
   		ans = (ans + a) % (1e9 + 7)
   	}
   	return ans
   }
   ```

   ```go
   func choose(n, k int) int {
   	if k >= n {
   		return 1
   	}
   	if k == 0 {
   		return 1
   	}
   	return (choose(n-1, k-1) + choose(n-1, k)) % (1e9 + 7)
   }
   ```

7. 但是，还是超时了，显然是在**计算组合数时**。由于是递推问题，显然可以使用 `DP` 进行优化和复用：
   $$
   c[n][k] = c[n-1][k-1] + c[n-1][k]
   $$

   ```go
   const (
   	mod  = 1e9 + 7
   	maxC = 13
   	maxV = 1e4 + 1
   )
   
   var c [maxV + maxC][maxC + 1]int
   
   func init() {
   	for i := range c {
   		c[i][0] = 1
   	}
   
   	for j := 1; j <= maxC; j++ {
   		for i := 1; i < len(c); i++ {
   			if j >= i {
   				c[i][j] = 1
   			}
   			c[i][j] = (c[i-1][j-1] + c[i-1][j]) % mod
   		}
   	}
   }
   ```

8. 最后的代码：

   ```go
   func idealArrays(n int, maxValue int) int {
   	ans := 0
   	for i := 1; i <= maxValue; i++ {
   		j := 2
   		tmp := i
   		a := 1
   		for tmp > 1 {
   			count := 0
   			for tmp%j == 0 {
   				count++
   				tmp /= j
   			}
   			if count > 0 {
   				a = (a * c[count+n-1][count]) % mod
   			}
   			j++
   		}
   		ans = (ans + a) % mod
   	}
   	return ans
   }
   ```

9. 终于搞定！



#### 排列组合例题

| ID   | LeetCode 题号                                                | 描述           | 思路          |
| ---- | ------------------------------------------------------------ | -------------- | ------------- |
| 1    | [2338. Count the Number of Ideal Arrays](https://leetcode.cn/problems/count-the-number-of-ideal-arrays/) | 理想数组的总数 | DP + 组合数学 |
|      |                                                              |                |               |
|      |                                                              |                |               |

