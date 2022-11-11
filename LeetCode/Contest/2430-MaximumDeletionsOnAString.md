## 2430. Maximum Deletions on a String

- [2430. Maximum Deletions on a String](https://leetcode.cn/problems/maximum-deletions-on-a-string/)

- [Weekly Contest 313 - Q4 Rating 2101](https://leetcode.com/contest/weekly-contest-313) 

#### 问题

You are given a string $s$ consisting of only lowercase English letters. In one operation, you can:

- Delete **the entire string** $s$, or

- Delete the **first** $i$ letters of $s$ if the first $i$ letters of $s$ are **equal to** the following $i$ letters in $s$, for any $i$ in the range $1 <= i <= s.length / 2$.

For example, if `s = "ababc"`, then in one operation, you could delete the first two letters of $s$ to get `"abc"`, since the first two letters of $s$ and the following two letters of $s$ are both equal to `"ab"`.

Return the **maximum** number of operations needed to delete all of $s$.

**Example 1:**

```
Input: s = "abcabcdabc"
Output: 2
Explanation:

- Delete the first 3 letters ("abc") since the next 3 letters are equal. Now, s = "abcdabc".
- Delete all the letters.
  We used 2 operations so return 2. It can be proven that 2 is the maximum number of operations needed.
  Note that in the second operation we cannot delete "abc" again because the next occurrence of "abc" does not happen in the next 3 letters.
```

**Example 2:**

```
Input: s = "aaabaab"
Output: 4
Explanation:

- Delete the first letter ("a") since the next letter is equal. Now, s = "aabaab".
- Delete the first 3 letters ("aab") since the next 3 letters are equal. Now, s = "aab".
- Delete the first letter ("a") since the next letter is equal. Now, s = "ab".
- Delete all the letters.
  We used 4 operations so return 4. It can be proven that 4 is the maximum number of operations needed.
```

**Example 3:**

```
Input: s = "aaaaa"
Output: 5
Explanation: In each operation, we can delete the first letter of s.
```

**Constraints:**

- $1 <= s.length <= 4000$
- $s$ consists only of lowercase English letters.



#### 分析

**序列 + 最值**，我们自然而然先从递推开始考虑，先考虑**样例2**的某个变形：`aabaab`，通过规则，我们有以下推导：

![2430-detail](../images/2430-detail.svg)

显然，这是可以递推的，我们可以写出推导公式：
$$
dp[i][j] = 
\begin{cases}
{dp[i+k][j]+1} & s[i:i+k]==s[i+k:i+2k] \\
\\
1 & others\\
\end{cases}
$$
考虑到题目中的规则，**我们完全可以删掉第二维**。

那么代码长得像这样：

```go
func deleteString(s string) int {
	dp := make([]int, len(s))
	for i := len(s) - 1; i >= 0; i-- {
		dp[i] = 1
		for j := 1; j <= (len(s)-i)/2; j++ {
			if s[i:i+j] == s[i+j:i+2*j] {
				dp[i] = max(dp[i], dp[i+j]+1)
			}
		}
	}
	return dp[0]
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```



**时间复杂度**：$O(n^2)$

**空间复杂度：**$O(n)$

