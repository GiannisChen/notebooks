## Math（数学）

- **梦回高一！数学赛高！**😀



#### 绝对值不等式

- 考虑以下公式的最小值，其中 $x_i$ 是数值， $a_i$ 是其权重，即重复出现的次数：
  $$
  \min \sum_{i=1}^{n} {a_i|x-x_i|} = ?
  $$
  **此处就有一个问题，当 $x$ 等于多少时，可以取到最小值？**

- 很容易就应该想到，两两配对的话，例如：
  $$
  |x-x_i|+|x-x_j| \qquad\qquad x_i \leq x_j
  $$
  那么当取 $x_i \leq x \leq x_j$ 时，上面这个式子是等于常数的。旋即我们可以递推一下，就能发现我们需要一个中位数。

  1. 当 $\sum a_i$ 是奇数时，我们只要定位到中位数就行了，然后算一下距离的加权；
  2. 当 $\sum a_i$ 不是奇数时，中位数通过左右两个数求平均值，这里我们只要任取**左右两个数组成的左闭右闭区间**中的一个数就行了；

- **例题**：[6216. Minimum Cost to Make Array Equal](https://leetcode.cn/problems/minimum-cost-to-make-array-equal/)



#### 素数筛

如果我们想要知道小于等于 $n$ 有多少个素数呢？一个自然的想法是对于小于等于 $n$ 的每个数进行一次质数检验。这种暴力的做法显然不能达到最优复杂度。

##### 埃拉托斯特尼筛法

考虑这样一件事情：对于任意一个大于 $1$ 的正整数 ，那么它的 $x$ $(x>1)$ 倍就是合数。利用这个结论，我们可以避免很多次不必要的检测。如果我们从小到大考虑每个数，然后同时把当前这个数的所有（比自己大的）倍数记为合数，那么运行结束的时候没有被标记的数就是素数了。

```go
isPrime := make([]int, n+1)
for i:=0; i<=n; i++ {
    isPrime = true
}
isPrime[0] = false
isPrime[1] = false
for i:=2; i*i<=n; i++ {
    if isPrime[i] {
        for j:=i*i; j<=n; j+=i { isPrime[j] = false }
    }
}
```



#### 数学的例题

| ID   | LeetCode 题号                                                | 描述                             | 思路                     |
| ---- | ------------------------------------------------------------ | -------------------------------- | ------------------------ |
| 1    | [6216. Minimum Cost to Make Array Equal](https://leetcode.cn/problems/minimum-cost-to-make-array-equal/) | 使用最小代价将数组变成同一个数   | 如上，绝对值不等式的应用 |
| 2    | [754. Reach a Number](https://leetcode.cn/problems/reach-a-number/) | 递增步长到达某一点的最小移动次数 | 分析 + 数学思维          |
| 3    | [6280. Closest Prime Numbers in Range](https://leetcode.cn/problems/closest-prime-numbers-in-range/) | 范围内最接近的两个质数           | 素数筛                   |

