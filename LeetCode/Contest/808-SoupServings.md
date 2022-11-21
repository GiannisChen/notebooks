## 808. Soup Servings

- [808. Soup Servings](https://leetcode.cn/problems/soup-servings/)

- [Weekly Contest 78 - Q3 Rating 2396](https://leetcode.com/contest/weekly-contest-78) 


#### 问题

- There are two types of soup: **type A** and **type B**. Initially, we have n ml of each type of soup. There are four kinds of operations:

  1. Serve `100 ml` of **soup A** and `0 ml` of **soup B**,

  2. Serve `75 ml` of **soup A** and `25 ml` of **soup B**,
  3. Serve `50 ml` of **soup A** and `50 ml` of **soup B**, 
  4. Serve `25 ml` of **soup A** and `75 ml` of **soup B**.

  When we serve some soup, we give it to someone, and we no longer have it. Each turn, we will choose from the four operations with an equal probability `0.25`. If the remaining volume of soup is not enough to complete the operation, we will serve as much as possible. We stop once we no longer have some quantity of both types of soup.

  > **Note** that we **do not** have an operation where all `100 ml`'s of **soup B** are used first.

- Return the probability that **soup A** will be empty first, plus half the probability that **A** and **B** become empty at the same time. Answers within $10^{-5}$ of the actual answer will be accepted.



**Example 1:**

```
Input: n = 50
Output: 0.62500

Explanation: If we choose the first two operations, A will become empty first.
For the third operation, A and B will become empty at the same time.
For the fourth operation, B will become empty first.
So the total probability of A becoming empty first plus half the probability that A and B become empty at the same time, is 0.25 * (1 + 1 + 0.5 + 0) = 0.625.
```

**Example 2:**

```
Input: n = 100
Output: 0.71875
```

**Constraints:**

$0 \leq n \leq 10^9$



#### 分析

- 一眼 **DP**：
  $$
  dp[i][j] = \frac{1}{4} \times (dp[i-4][j] + dp[i-3][j-1] + dp[i-2][j-2] + dp[i-1][j-3])
  $$
  由于每次分配的单位的最大公约数为 $25$ ，所以可以简化为上面的递推方程。同时，考虑边界条件：

  1. 当 $i=0,j=0$ 时，显然满足 **AB** 同时分完，因此是 $1 \times 0.5$ ；
  2. 当 $i=0,j>0$ 时，显然满足 **A** 先分完，因此是 $1$ ；
  3. 当 $i>0,j=0$ 时，显然满足 **B** 先分完，因此是 $1 \times 0$ ；

- 第二眼瞄到限制条件，$10^9$ 打扰了。打开题解，发现赫然是 **DP** ，再细看，竟然可以使用浮点数精度做范围放缩：

  当 $n \ge 4475$ 时，所求概率已经大于 $0.99999$ 了（可以通过上面的动态规划方法求出），它和 $1$ 的误差（无论是绝对误差还是相对误差）都小于 $10^{-5}$ 。 实际我们在进行测算时发现：
  $$
  n = 4475, \quad p \approx 0.999990211307 \\ 
  n = 4476, \quad p \approx 0.999990468596 \\
  $$
  因此在 $n \ge 179 \times 25$ 时，我们只需要输出 $1$ 作为答案即可。在其它的情况下，我们使用动态规划计算出答案。

- ~~只能说，不愧是 $2396.6$ 的题，挂尼玛的 **mid** 呢？~~



**时间复杂度**：$O(C^2)$

**空间复杂度：**$O(C^2)$