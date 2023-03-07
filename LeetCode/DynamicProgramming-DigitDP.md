## 数位DP

数位DP，在看似不可能的复杂度下简化。

#### 参考

- https://oi-wiki.org/dp/number/
- https://github.com/EndlessCheng/codeforces-go/blob/master/copypasta/dp.go#L1918
- https://leetcode.cn/problems/count-special-integers/solution/shu-wei-dp-mo-ban-by-endlesscheng-xtgx/



数位DP往往是在一定限制下找到满足条件的数字，或者更加常见的，找到满足某种要求的数字数目，通常被称作“好数”。这类数字往往很大，哪怕线性过滤，都有可能超时（时间复杂度远大于$O(10^7)$）。

那么，**数位**DP中的数位因此而来，我们不需要遍历数字本身，而是一位一位地去看，通过遍历，以及状态压缩，得到最终地结果。

**难点**：

数位DP另开一章详细讲述，是因为数位DP一定会面对的条件：**上限**。数位DP的来源就是，状态复用，3999和4999是不同的数字，但\*999是一样的，自然而然，这里就存在了递推。

以[2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/)为例子，如果一个正整数每一个数位都是**互不相同**的，我们称它是**特殊整数** 。给你一个**正**整数 `n` ，请你返回区间 `[1, n]` 之间特殊整数的数目。

我们分析一下，当我们在选择**ABCDE**这五位数的**A**位时，我们并不需要知道**BCDE**到底是怎么排序的，只要知道，我们拿了**{B, C, D, E}**，也许是**BDCE**，也许是**BECD**，anyway，都不影响**A**的选择，那么，这里就有状态压缩的可取之处了，我们写一下状态转移方程：
$$
dp[i][mask \, | \, 1 \ll j] += dp[i+1][mask]
$$
其中，$mask$是数字的按位枚举表示，例如$0000000001$表示取一个0，$0000010001$表示取了一个0还娶了一个4。

看着简单吧，其实不是那么容易的。我们还有两个问题：

1. 上限如何解决
2. 前导零的问题

我们首先解决一下上限的问题，其实很直白：

- 当前数位遇到上限了，看向下一个数位，下一个数位也只能用到限制位$k$到$0$这几个数了，用到$k$，就是递归了，不用$k$，那么在下一个数位就可以用缓存的内容。上面提到的递归，最后体现为上限这一个数了。

那么前导零该怎么处理呢：

- 前导零本质上是没有数字的，不用选，但是由于我们从最高位开始递推，所以会有这个问题，我们借鉴了）**0X3F**大神的思路，在递推中设置一个符号位表示前导零，那么，每次存在前导零，即当前高位全为0的时候，可以选择大于零的数字，那么从该状态中脱出，要么继续啥都不选，以前导零的状态继续递推，最后的结果就是一个零蛋，看情况需要返回啥。

#### 代码

```go
func countSpecialNumbers(n int) int {
	s := strconv.Itoa(n)
	m := len(s)
	nums := make([]int, m)
	for i := 0; i < m; i++ {
		nums[i] = int(s[i] - '0')
	}

	dp := make([][1 << 10]int, m)
	for i := 0; i < m; i++ {
		for j := 0; j < 1<<10; j++ {
			dp[i][j] = -1
		}
	}

	var f func(i, mask int, leadingZeros, isLimit bool) int
	f = func(i, mask int, leadingZeros, isLimit bool) (res int) {
		if i >= m {
			if !leadingZeros {
				return 1
			}
			return 0
		}
		if !isLimit && !leadingZeros && dp[i][mask] > -1 {
			return dp[i][mask]
		}

		if leadingZeros {
			res += f(i+1, mask, true, false)
		}

		start, end := 1, 1<<9
		if leadingZeros {
			start = 1 << 1
		}
		if isLimit {
			end = 1 << nums[i]
		}

		for j := start; j <= end; j = j << 1 {
			if mask&j == 0 {
				res += f(i+1, mask|j, false, isLimit && j == end)
			}
		}
		dp[i][mask] = res
		return res
	}

	ans := f(0, 0, true, true)
	return ans
}
```



#### 其他例题

| ID   | LeetCode 题号                                                | 描述                                 | 思路                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------ | ------------------------------------------------------------ |
| 1    | [2376. Count Special Integers](https://leetcode.cn/problems/count-special-integers/) | 正整数每一个数位都是 **互不相同** 的 | 二维状态压缩，数位枚举表示取数                               |
| 2    | [357. Count Numbers with Unique Digits](https://leetcode.cn/problems/count-numbers-with-unique-digits/) | 各位数字都不同的数字                 | 2376的特殊情况                                               |
| 3    | [233. Number of Digit One](https://leetcode.cn/problems/number-of-digit-one/) | 非负整数中数字 `1` 出现的个数        | 同样二维状态压缩，统计高位已经出现过的`1`的个数，然后统计接下来的 |
| 4    | [面试题 17.06. Number Of 2s In Range LCCI](https://leetcode.cn/problems/number-of-2s-in-range-lcci/) | 从 0 到 n (含 n) 中数字 2 出现的次数 | 同上                                                         |
| 5    | [1012. Numbers With Repeated Digits](https://leetcode.cn/problems/numbers-with-repeated-digits/) | **至少 1 位** 重复数字的正整数的个数 | 2376的取逆                                                   |
| 6    | 248                                                          |                                      |                                                              |
| 7    | 600                                                          |                                      |                                                              |
| 8    | 788                                                          |                                      |                                                              |
| 9    | 902                                                          |                                      |                                                              |
| 10   | 1088                                                         |                                      |                                                              |
| 11   | 1215                                                         |                                      |                                                              |
| 12   | 1397                                                         |                                      |                                                              |

