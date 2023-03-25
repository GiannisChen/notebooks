## 树状数组

- https://oi-wiki.org/ds/fenwick/#%E6%A0%91%E7%8A%B6%E6%95%B0%E7%BB%84



普通树状数组维护的信息及运算要满足**结合律**且**可差分**，如加法（和）、乘法（积）、异或等。

- 结合律：$(x \circ y) \circ z = x \circ (y \circ z)$，其中$\circ$是一个二元运算符。
- 可差分：具有逆运算的运算，即已知$x \circ y$和$x$可以求出$y$。

事实上，树状数组能解决的问题是线段树能解决的问题的子集：树状数组能做的，线段树一定能做；线段树能做的，树状数组不一定可以。然而，树状数组的代码要远比线段树短，时间效率常数也更小，因此仍有学习价值。

有时，在差分数组和辅助数组的帮助下，树状数组还可解决更强的**区间加单点值**和**区间加区间和**问题。



### 例子

- 前缀和：$a[1 \dots 7]$，$a_1+a_2+a_3+ \cdots + a_7$是一种方案，同样我们也可以使用区间求和。

那么我们应用这样一个数据结构：

![img](images/fenwick.svg)



### lowbit

`lowbit(x) = x & -x`，将 `x` 的二进制所有位全部取反，再加 1，就可以得到 `-x` 的二进制编码。例如，![6](data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7) 的二进制编码是 `110`，全部取反后得到 `001`，加 `1` 得到 `010`。

设原先 `x` 的二进制编码是 `(...)10...00`，全部取反后得到 `[...]01...11`，加 `1` 后得到 `[...]10...00`，也就是 `-x` 的二进制编码了。这里 `x` 二进制表示中第一个 `1` 是 `x` 最低位的 `1`。

`(...)` 和 `[...]` 中省略号的每一位分别相反，所以 `x & -x = (...)10...00 & [...]10...00 = 10...00`，得到的结果就是 `lowbit`。

就这样，`lowbit`取到了`x`中的最低位的`1`，我需要的，是取遍`x`中所有的`1`，因此有查询：



### 区间查询

查询$a[1 \dots x]$：

```go
for x != 0 {
	ans += c[x]
    x -= lowbit(x)
}
```

 

### 单点修改

修改$a[x]$：

```go
for x != 0 {
    c[x] = update(c[x], v)
    x -= lowbit(x)
}
```



### 建树

预处理：（自底向上，`x += lowbit(x)`）

复杂度$O(n \cdot log\,n)$



### 裸题

- [307. Range Sum Query - Mutable](https://leetcode.cn/problems/range-sum-query-mutable/)

```go
type NumArray struct {
	nums []int
	c    []int
	n    int
}

func Constructor(nums []int) (res NumArray) {
	res = NumArray{nums: nums, n: len(nums)}
	res.c = make([]int, res.n+1)
	for i, num := range nums {
		for x := i + 1; x <= res.n; x += lowbit(x) {
			res.c[x] += num
		}
	}
	return res
}

func lowbit(x int) int {
	return x & (-x)
}

func (this *NumArray) find(index int) int {
	ret := 0
	for x := index + 1; x > 0; x -= lowbit(x) {
		ret += this.c[x]
	}
	return ret
}

func (this *NumArray) Update(index int, val int) {
	diff := val - this.nums[index]
	for x := index + 1; x <= this.n; x += lowbit(x) {
		this.c[x] += diff
	}
	this.nums[index] = val
}

func (this *NumArray) SumRange(left int, right int) int {
	return this.find(right) - this.find(left-1)
}
```



### 最长递增子序列DP优化

- [300. Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/)

$$
dp[i] = \max(dp[i], dp[j])
$$

很抽象的更新方程，因为正常情况下我们要遍历$j<i$内所有小于`nums[i]`的`nums[j]`对应的`dp[j]`，我们可以用树状数组优化他：

```go
type Fenwick struct {
	dp []int
	c  []int
	n  int
}

func lengthOfLIS(nums []int) int {
	ans := 0
	m := map[int]int{}
	for _, num := range nums {
		m[num] = 0
	}

	newNums := make([]int, len(m))
	newNums = newNums[:0]
	for i := range m {
		newNums = append(newNums, i)
	}
	sort.Ints(newNums)
	for i, num := range newNums {
		m[num] = i + 1
	}

	f := Fenwick{
		dp: make([]int, len(newNums)+1),
		c:  make([]int, len(newNums)+1),
		n:  len(newNums),
	}

	for _, num := range nums {
		tmp := 0
		i := m[num]
		for x := i - 1; x > 0; x -= lowbit(x) {
			tmp = max(tmp, f.c[x])
		}
		tmp++
		f.dp[i] = tmp
		for x := i; x <= f.n; x += lowbit(x) {
			f.c[x] = max(f.c[x], tmp)
		}
		ans = max(ans, tmp)
	}
	return ans
}

func lowbit(x int) int {
	return x & -x
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

`离散化`+`DP`



### 最长“递增”子序列DP优化

- [1626. Best Team With No Conflicts](https://leetcode.cn/problems/best-team-with-no-conflicts/)

略，比上面稍稍复杂一点点



