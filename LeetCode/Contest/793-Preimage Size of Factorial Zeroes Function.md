## 793. Preimage Size of Factorial Zeroes Function

- [793. Preimage Size of Factorial Zeroes Function](https://leetcode.cn/problems/preimage-size-of-factorial-zeroes-function/)

- [Weekly Contest 74 - Q4 Rating 2100](https://leetcode.com/contest/weekly-contest-74) 

#### 问题

- Let $f(x)$ be the number of zeroes at the end of $x!$. Recall that $x! = 1 * 2 * 3 * ... * x$ and by convention, $0! = 1$.

  - For example, $f(3) = 0$ because $3! = 6$ has no zeroes at the end, while $f(11) = 2$ because $11! = 39916800$ has two zeroes at the end.


- Given an integer $k$, return the number of non-negative integers $x$ have the property that $f(x) = k$.

**Example 1:**

```
Input: k = 0
Output: 5
Explanation: 0!, 1!, 2!, 3!, and 4! end with k = 0 zeroes.
```

**Example 2:**

```shell
Input: k = 5
Output: 0
Explanation: There is no x such that x! ends in k = 5 zeroes.
```

**Example 3:**

```shelll
Input: k = 3
Output: 5
```

**Constraints:**

- $0 <= k <= 10^9$



#### 分析

本题的前置问题是 [172. Factorial Trailing Zeroes](https://leetcode.cn/problems/factorial-trailing-zeroes/) ，我们先来看一下这道题：

Given an integer $n$, return the number of trailing zeroes in $n!$.

Note that $n! = n * (n - 1) * (n - 2) * ... * 3 * 2 * 1$.

**Example 1:**

```
Input: n = 3
Output: 0
Explanation: 3! = 6, no trailing zero.
```

**Example 2:**

```
Input: n = 5
Output: 1
Explanation: 5! = 120, one trailing zero.
```

**Example 3:**

```
Input: n = 0
Output: 0
```

**Constraints:**

$0 <= n <= 10^4$

---

我们不难发现，对一个阶乘数字**末尾零**的个数做出贡献的必然是 $5$ 和 $2$ ，对于任意一个阶乘中出现的 $5$ 的倍数，必然有超过因子 $5$ 个数的因子 $2$ ，出现，这个推论很简单，因为 $4<5$ 。那么我们只要考虑 **$k!$ 里出现了多少个因子 $5$** 。

![172-details](../images/172-details.svg)

在数字中找个规律，不难发现，我们只要统计以 $[5,5^2,5^3,\cdots,5^i]$ 的因子 个数就行，由于 $5^2$ 本身就是 $5$ 的倍数，所以得到以下规律： 含 $5$ 这个因子的数的个数是 $\lfloor k\div5 \rfloor$，以此类推，含 $5^2$ 的个数是  $\lfloor {\lfloor k/5 \rfloor} /5 \rfloor$ ……

那么答案就很简单了：

```go
var zeros = func(n int) (res int) {
    for n >= 5 {
        res += n / 5
        n /= 5
    }
    return
}
```

让我们回到**本体793题**，显然这个一个逆过程，给定了 $zeros$ ，要推导满足这个  $zeros$ 的 $k$ 的个数，很容易发现，答案不是  $0$ 就是  $5$ 。从一个  $5$  的倍数到下一个 $5$ 的倍数正好有 $5$ 个数，如果 $zeros$ 并不是合法的末尾零个数，则返回  $0$ 。

一个很朴素的思路摆在我们面前，**二分查找**。但是，如何确定二分查找的边界呢，下界很好确定，设置为 $0$ 就行了，上界和 $zeros$ 有关，这又需要一次找规律，但是不难发现，我们只要知道此时 $zeros$ 的合法性，那么不妨找 $5$ 的倍数来算，~~干净又卫生~~。这让我们又发现了一个小细节，也就是根据**172题**的计算公式发现， $zeros <=5 \times k$ ，那么，上界也定下来了。

```go
if search(0, 5*k) {
    return 5
}
return 0
```

最后加上二分，**AC代码**就是：

```go
package main

func preimageSizeFZF(k int) int {
	var search func(l, r int) bool
	var zeros = func(n int) (res int) {
		for n >= 5 {
			res += n / 5
			n /= 5
		}
		return
	}
	search = func(l, r int) bool {
		if l > r {
			return false
		}
		mid := l + (r-l)/2
		if z := zeros(mid); z == k {
			return true
		} else if z > k {
			return search(l, mid-1)
		} else {
			return search(mid+1, r)
		}
	}
	if search(0, 5*k) {
		return 5
	}
	return 0
}
```



**时间复杂度**：$O(log^2\,k) \qquad k \; for \; zeros$

**空间复杂度：**$O(1)$