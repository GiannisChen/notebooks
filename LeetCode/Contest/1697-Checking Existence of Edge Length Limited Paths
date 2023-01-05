## 1697. Checking Existence of Edge Length Limited Paths

- [1697. Checking Existence of Edge Length Limited Paths](https://leetcode.cn/problems/checking-existence-of-edge-length-limited-paths/)

- [Weekly Contest 220 - Q4 Rating 2300](https://leetcode.com/contest/weekly-contest-220) 


#### 分析

看到一个点 $A$ 到另一个点 $B$ 之间所有的路径是否有超过限制的距离的，第一反应是 **dijkstra** 或者 **floyd** 算法，发现并不是那么回事，题目并不关心整条路是怎么样的，只关心点对点。

那么 **并查集** 就该登场了，但是这是一个典型的多查询问题，涉及到离线算法，那么我们需要设计一定的并查集流程来完成他。不难发现，当 $A$ 到 $B$ 存在这么一条通路时，每个连通都是小于这个限制的，我们只要将 `edgeList` 中所有小于该限制的路认为是通路，其他的将其断路，则通过并查集就能找到是否有这么一条通路。

接下来就是迭代和递推了，巧的是，当限制大小增长时，总是包含限制较小的结果，我们并不需要重构并查集。

**接下来是代码**：

```go
func distanceLimitedPathsExist(n int, edgeList [][]int, queries [][]int) []bool {
	queriesIdx := make([]int, len(queries))
	for i := range queriesIdx {
		queriesIdx[i] = i
	}

	sort.Slice(queriesIdx, func(i, j int) bool {
		return queries[queriesIdx[i]][2] < queries[queriesIdx[j]][2]
	})
	sort.Slice(edgeList, func(i, j int) bool {
		return edgeList[i][2] < edgeList[j][2]
	})

	fa := make([]int, n)
	for i := 0; i < n; i++ {
		fa[i] = i
	}

	find := func(node int) int {
		idx := node
		for fa[idx] != idx {
			idx = fa[idx]
		}
		fa[node] = idx
		return idx
	}

	k := 0
	m := len(edgeList)
	ans := make([]bool, len(queries))
	for _, idx := range queriesIdx {
		for k < m && edgeList[k][2] < queries[idx][2] {
			if a, b := find(edgeList[k][0]), find(edgeList[k][1]); a != b {
				if b > a {
					a, b = b, a
				}
				fa[a] = b
			}
			k++
		}
		ans[idx] = find(queries[idx][0]) == find(queries[idx][1])
	}
	return ans
}
```



我们认为 `edgeList` 数量为 $e$ ，`queries` 为 $m$ ，则：

- **时间复杂度**：$O(e \cdot log\,e+m \cdot log\,m+(e+m)log\,n+n)$

- **空间复杂度：**$O(log\,e + m + n)$
