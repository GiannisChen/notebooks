## 6260. Maximum Number of Points From Grid Queries

- [6260. Maximum Number of Points From Grid Queries](https://leetcode.cn/problems/maximum-number-of-points-from-grid-queries/)
- [Weekly Contest 323 - Q4 Rating ?](https://leetcode.com/contest/weekly-contest-323) 



#### 问题

You are given an $m \times n$ integer matrix `grid` and an array `queries` of size $k$.

Find an array `answer` of size $k$ such that for each integer `queres[i]` you start in the **top left** cell of the matrix and repeat the following process: 

- If `queries[i]` is **strictly** greater than the value of the current cell that you are in, then you get one point if it is your first time visiting this cell, and you can move to any **adjacent** cell in all $4$ directions: up, down, left, and right.
- Otherwise, you do not get any points, and you end this process.

After the process, `answer[i]` is the **maximum** number of points you can get. **Note** that for each query you are allowed to visit the same cell **multiple** times.

Return the resulting array `answer`.

 

**Example 1:**

![img](https://assets.leetcode.com/uploads/2022/10/19/yetgriddrawio.png)

```
Input: grid = [[1,2,3],[2,5,7],[3,5,1]], queries = [5,6,2]
Output: [5,8,1]
Explanation: The diagrams above show which cells we visit to get points for each query.
```

**Example 2:**

![img](https://assets.leetcode.com/uploads/2022/10/20/yetgriddrawio-2.png)

```
Input: grid = [[5,2,1],[1,1,2]], queries = [3]
Output: [0]
Explanation: We can not get any points because the value of the top left cell is already greater than or equal to 3.
```

**Constraints:**

- $m == grid.length$
- $n == grid[i].length$
- $2 <= m, n <= 1000$
- $4 <= m \times n <= 10^5$
- $k == queries.length$
- $1 <= k <= 10^4$
- $1 <= grid[i][j], queries[i] <= 10^6$



#### 分析

看到图，我抬手就是一个遍历，但是对于查询该怎么做呢？通过看规则，我们知道，每次查询都是以 `grid[0][0]` 作为起点，能到达标号小于 `queries[i]` 的最大面积。每次查询都需要走一遍？其实不然，我们不难发现，标号较大的可达面积**必定**覆盖标号较小的。

那么，我们采用 **BFS** ，稍稍改变一下遍历规则：不妨设当前标号为 `q` , 当存在比当前标号 `q` 小的地区 `grid[i][j]` 时，我们优先遍历该区域，并用**队列**缓存他的相邻，此时，标号大的地区就是一堵临时的“墙”。当队列中所有地区的标号都是大于当前标号时，我们更新 `q` 为队列中最小地区标号，继续上述过程。每次更新 `q` 时，对上一次的结果进行缓存，所得到的数对序列总是根据标号 `q` 升序的，这让我们的查询可以用**二分查找**。

显然，这里的队列是**优先队列**，为了每次遍历都是标号最小的地区。



```go
var surround = [][]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}}

type Item struct {
	i, j, val int
}

type Heap []*Item

func (h Heap) Len() int {
	return len(h)
}

func (h Heap) Less(i, j int) bool {
	return h[i].val < h[j].val
}

func (h *Heap) Swap(i, j int) {
	(*h)[i], (*h)[j] = (*h)[j], (*h)[i]
}

func (h *Heap) Push(x interface{}) {
	*h = append(*h, x.(*Item))
}

func (h *Heap) Pop() interface{} {
	old := *h
	top := old[len(old)-1]
	*h = old[:len(old)-1]
	return top
}

func maxPoints(grid [][]int, queries []int) []int {
	n, m := len(grid), len(grid[0])

	ansIdx := make([]int, 0)
	ansVal := make([]int, 0)

	var h *Heap = &Heap{}
	heap.Push(h, &Item{i: 0, j: 0, val: grid[0][0]})

	count := grid[0][0] + 1
	grid[0][0] = -1

	sum := 0
	for h.Len() > 0 {
		item := heap.Pop(h).(*Item)

		if item.val >= count {
			ansIdx = append(ansIdx, count)
			ansVal = append(ansVal, sum)
			count = item.val + 1
			sum++
		} else {
			sum++
		}
		for _, s := range surround {
			if _i, _j := item.i+s[0], item.j+s[1]; _i >= 0 && _i < n && _j >= 0 && _j < m && grid[_i][_j] != -1 {
				heap.Push(h, &Item{
					i:   _i,
					j:   _j,
					val: grid[_i][_j],
				})
				grid[_i][_j] = -1
			}
		}
	}

	if len(ansIdx) == 0 || count != ansIdx[len(ansIdx)-1] {
		ansIdx = append(ansIdx, count)
		ansVal = append(ansVal, sum)
	}

	ans := make([]int, 0)
	for _, query := range queries {
		i := sort.SearchInts(ansIdx, query)
		if i >= len(ansIdx) || ansIdx[i] != query {
			i--
		}
		if i < 0 {
			ans = append(ans, 0)
		} else {
			ans = append(ans, ansVal[i])
		}
	}
	return ans
}
```

你问代码为什么这么长？问就是 **“大道至简”的Golang**。



**时间复杂度**：$O(n^2)$

**空间复杂度：**$O(n)$

