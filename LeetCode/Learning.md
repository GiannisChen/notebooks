# LeetCode

![img](images/hello_leetcode.png)

### 题目列表

- https://www.1point3acres.com/bbs/thread-840864-1-1.html

### 参考

- https://books.halfrost.com/leetcode/
- https://algorithm-essentials.soulmachine.me/

## 图 Graph

- 图的基础就是**图的遍历**，树或者森林也是特殊的图，遍历方法通常就是 `DFS` **深度遍历**和 `BFS` **广度遍历**。

#### 图的遍历

-  `Graph Traverse`

##### 通用结构

```go
type graph [][]int
type visited [][]bool
var surrounds [][]int = [][]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}}

func initGraph(n, m int) (g graph) {
	g = make([][]int, n)
	for i := range g {
		g[i] = make([]int, m)
	}
	return
}

func initVisited(n, m int) (v visited) {
	v = make([][]bool, n)
	for i := range v {
		v[i] = make([]bool, m)
	}
	return
}
```

##### BFS

```go
func BFS(g graph, n, m int) {
	q := make([]int, 0)
	v := initVisited(n, m)

	bfs := func(i, j int) {
		q = append(q, i, j)
		for len(q) > 0 {
			i, j := q[0], q[1]
			v[i][j] = true
			for _, surround := range surrounds {
				if _i, _j := i+surround[0], j+surround[1]; _i >= 0 && _i < n && _j >= 0 && _j < m &&
					!v[_i][_j] && g[_i][_j] != 0 {
					q = append(q, _i, _j)
					v[_i][_j] = true
				}
			}
			q = q[2:]
		}
	}

	for i := 0; i < n; i++ {
		for j := 0; j < m; j++ {
			if !v[i][j] && g[i][j] != 0 {
				bfs(i, j)
			}
		}
	}
}
```

##### DFS

```go
func DFS(g graph, n, m int) {
	v := initVisited(n, m)

	var dfs func(int, int)
	dfs = func(i, j int) {
		v[i][j] = true
		for _, surround := range surrounds {
			if _i, _j := i+surround[0], j+surround[1]; _i >= 0 && _i < n && _j >= 0 && _j < m &&
				!v[_i][_j] && g[_i][_j] != 0 {
				dfs(_i, _j)
			}
		}
	}

	for i := 0; i < n; i++ {
		for j := 0; j < m; j++ {
			if !v[i][j] {
				dfs(i, j)
			}
		}
	}
}
```

##### 图的遍历例题

1. 常规的 `node-edge` 图，通过建立**邻接矩阵**或者**邻接表**（`ID-2`，`ID-7`，`ID-9`）然后遍历；
2. 输入就是二维矩阵，可以看成是图，通常需要按规则遍历四周（`ID-1`，`ID-3`，`ID-4`，`ID-5`，`ID-6`，`ID-8`），同时也要注意是否访问规则，`ID-1` 共享同一个访问标志矩阵，而 `ID-3` 需要维护能够回溯的标志；
3. 把状态看成是 `node` 而状态转化看成是 `edge` ，转化为类似动态规划的问题，例如 `ID-7`，`ID-9`；

| ID   | LeetCode 题号                                                | 描述                                  | 思路ID                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------- | ------------------------------------------------------------ |
| 1    | [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/) | 相互不连接的岛屿的个数                | 直接 `DFS` 或者 `BFS` ，标记访问过的位置                     |
| 2    | [690. Employee Importance](https://leetcode.cn/problems/employee-importance/) | 递归返回自身和下属重要性之和          | 广义图论，`DFS` 或 `BFS` ，甚至不用建图                      |
| 3    | [79. Word Search](https://leetcode.cn/problems/word-search/) | 在字母表里找可能出现的字符串          | `DFS`，由于字母表最大容量较小，采用位运算来记录是否访问过，而不是矩阵 |
| 4    | [1254. Number of Closed Islands](https://leetcode.cn/problems/number-of-closed-islands/) | 四周都被水包围的岛屿个数              | `ID-1` 的变种，只需把边界一圈先遍历掉，但不计数即可          |
| 5    | [1905. Count Sub Islands](https://leetcode.cn/problems/count-sub-islands/) | 第二张图的岛屿必须建在第一个岛屿上    | 需要遍历完，不然会产生额外的岛，其他没啥和 `ID-4` 一样       |
| 6    | [417. Pacific Atlantic Water Flow](https://leetcode.cn/problems/pacific-atlantic-water-flow/) | 不同海拔的海岛是否可达上下两侧        | `DFS` 反向操作，从边上向中心传播                             |
| 7    | [127. Word Ladder](https://leetcode.cn/problems/word-ladder/) | 单词接龙，每次只能变化一个字符 `hard` | `BFS` ，邻接表，用双向队列控制扇出                           |
| 8    | [695. Max Area of Island](https://leetcode.cn/problems/max-area-of-island/) | 返回面积最大的岛的面积                | `DFS` 多一个返回值，很直白                                   |
| 9    | [1345. Jump Game IV](https://leetcode.cn/problems/jump-game-iv/) | 按规则跳跃                            | `DFS`，我用了双队列优化，通过删邻接表来控制队列规模          |



#### 拓扑问题

- **拓扑排序**（`Topological Sort`）

##### 关键路径问题（先后顺序）

- 这种题从入度来考虑，有两种思路：（以 `ID-1` 为例，`ID-2` 同理）

  1. `DFS` + 入度，利用**队列**来完成遍历，每次更新节点入度，比较直白；

     ```go
     func findOrder(numCourses int, prerequisites [][]int) (ans []int) {
     	degrees := make([]int, numCourses)
     	ans = make([]int, 0)
     	linkList := make([][]int, numCourses)
     
     	for _, prerequisite := range prerequisites {
     		a, b := prerequisite[0], prerequisite[1]
     		degrees[a]++
     		linkList[b] = append(linkList[b], a)
     	}
     
     	queue := make([]int, 0)
     	for i, degree := range degrees {
     		if degree == 0 {
     			queue = append(queue, i)
     		}
     	}
     
     	for len(queue) > 0 {
     		q := queue[0]
     		queue = queue[1:]
     		ans = append(ans, q)
     		for _, next := range linkList[q] {
     			degrees[next]--
     			if degrees[next] == 0 {
     				queue = append(queue, next)
     			}
     		}
     	}
     	if len(ans) < numCourses {
     		return nil
     	}
     	return ans
     }
     ```

  2. `BFS` + 访问控制数组，通过递归到出度为零或者下一个节点都已访问过的节点为止，然后翻转；本质上是利用**栈**来完成遍历；

     ```go
     func findOrder(numCourses int, prerequisites [][]int) []int {
     	linkList := make([][]int, numCourses)
     	visits := make([]int, numCourses)
     	ans := make([]int, 0)
     
     	var dfs func(int) bool
     	dfs = func(node int) bool {
     		visits[node] = 1
     		for _, next := range linkList[node] {
     			if visits[next] == 0 {
     				if !dfs(next) {
     					return false
     				}
     			} else if visits[next] == 1 {
     				return false
     			}
     		}
     		visits[node] = 2
     		ans = append(ans, node)
     		return true
     	}
     
     	for _, prerequisite := range prerequisites {
     		linkList[prerequisite[1]] = append(linkList[prerequisite[1]], prerequisite[0])
     	}
     	for i := 0; i < numCourses; i++ {
     		if visits[i] == 0 && !dfs(i) {
     			return nil
     		}
     	}
     
     	if len(ans) != numCourses {
     		return nil
     	}
     	for i := 0; i < numCourses/2; i++ {
     		ans[i], ans[numCourses-1-i] = ans[numCourses-1-i], ans[i]
     	}
     	return ans
     }
     ```

##### 成环问题

1. **有向图**和之前先后顺序问题一样的思路，可以通过：（以 `ID-3` 为例）

   1. `DFS` 和遍历状态表来判断是否成环，如果遍历过程中碰到状态时遍历中的时候，就是有环了；
   2. `BFS` 层次遍历是否出现没遍历过但是入度为零不存在；

2. **无向图**的**割点**与**桥**的问题 —— **Tarjan算法**：

   **Tarjan算法**是基于 `DFS` 的算法，用于求解图的连通性问题。**Tarjan算法**可以在线性时间内求出无向图的割点与桥，进一步地可以求解**无向图的双连通分量**；同时，也可以求解**有向图的强连通分量**、**必经点与必经边**。

   - **割点**：若从图中删除节点 x 以及所有与 x 关联的边之后，图将被分成两个或两个以上的不相连的子图，那么称 x 为图的**割点**：（图中 `1`、`6`）

     ```go
     package main
     
     import (
        "fmt"
        "sort"
     )
     
     func main() {
        var n, m int
        fmt.Scanln(&n, &m)
        linkList := make([][]int, n+1)
        var a, b int
        for m > 0 {
           m--
           fmt.Scanln(&a, &b)
     
           linkList[a] = append(linkList[a], b)
           linkList[b] = append(linkList[b], a)
        }
     
        dfn := make([]int, n+1)
        low := make([]int, n+1)
        ans := make([]int, 0)
        clock := 0
     
        var tarjan func(int, int)
        tarjan = func(cur, fa int) {
           clock++
           dfn[cur], low[cur] = clock, clock
           child := 0
           for _, next := range linkList[cur] {
              if dfn[next] == 0 {
                 child++
                 tarjan(next, cur)
                 low[cur] = min(low[cur], low[next])
                 if fa != cur && low[next] >= dfn[cur] {
                    ans = append(ans, cur)
                 }
              } else if next != fa {
                 low[cur] = min(low[cur], dfn[next])
              }
           }
           if fa == cur && child >= 2 {
              ans = append(ans, cur)
           }
        }
     
        for i := 1; i <= n; i++ {
           if dfn[i] == 0 {
              tarjan(i, i)
              clock = 0
           }
        }
     
        fmt.Print(len(ans))
        if len(ans) == 0 {
           return
        }
        sort.Ints(ans)
        fmt.Println()
        for i, an := range ans {
           if i == 0 {
              fmt.Printf("%d", an)
           } else {
              fmt.Printf(" %d", an)
           }
        }
        return
     }
     
     func min(a, b int) int {
        if a < b {
           return a
        }
        return b
     }
     ```
   
     ![cut-point-graph](images/cut-point-graph.svg)
   
   - **桥**：若从图中删除边 e 之后，图将分裂成两个不相连的子图，那么称 e 为图的**桥**或**割边**：（红边）
   
     ![bridge-edge-graph](images/bridge-edge-graph.svg)
   
     ```go
     func criticalConnections(n int, connections [][]int) (ans [][]int) {
        linkList := make([][]int, n)
        dfn := make([]int, n)
        low := make([]int, n)
        ts := 0
     
        for _, c := range connections {
           linkList[c[0]] = append(linkList[c[0]], c[1])
           linkList[c[1]] = append(linkList[c[1]], c[0])
        }
     
        var tarjan func(int, int)
        tarjan = func(i int, fa int) {
           ts++
           dfn[i], low[i] = ts, ts
           for _, next := range linkList[i] {
              if next == fa {
                 continue
              }
              if dfn[next] == 0 {
                 tarjan(next, i)
                 low[i] = min(low[next], low[i])
                 if dfn[i] < low[next] {
                    ans = append(ans, []int{i, next})
                 }
              } else {
                 low[i] = min(low[next], low[i])
              }
           }
        }
        for i := 0; i < n; i++ {
           if dfn[i] == 0 {
              tarjan(i, -1)
           }
        }
        return
     }
     ```



##### 图染色问题

- **二分图**：（`ID-5`）
  1. 当且仅当图中无**奇圈**，与之前 `DFS` 一样，但是访问中的点标记不太一样，仿照 `tarjan` 算法中的 `clock` ，我们也能给环计数，本质上是**图论**；
  2. 模拟染色，比上面的方法更加直白，通过父亲节点判断下一个颜色，如果染色冲突则跳出，那么，`BFS` 和 `DFS` 皆可，本质上是**模拟** + **遍历**；

##### 拓扑问题例题

| ID   | LeetCode 题号                                                | 描述                                                 | 思路                                                         |
| ---- | ------------------------------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------ |
| 1    | [210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/) | 经典图论中先后顺序问题                               | `BFS` `DFS` 都可以，注意谁先谁后就行，可以通过去入度时候进队列来判断是否为入度零 |
| 2    | [269. Alien Dictionary](https://leetcode.cn/problems/alien-dictionary/) | 给定一个按字典升序排列的字符串数组，判定字符先后顺序 | 按照规则构建对应先后顺序表就行，小细节                       |
| 3    | [207. Course Schedule](https://leetcode.cn/problems/course-schedule/) | 图论中是否成环的问题                                 | `DFS` + 状态数组                                             |
| 4    | [1192. Critical Connections in a Network](https://leetcode.cn/problems/critical-connections-in-a-network/) | 图中是否有桥                                         | 判断是否有桥，板子 `tarjan` 算法                             |
| 5    | [785. Is Graph Bipartite?](https://leetcode.cn/problems/is-graph-bipartite/) | 是否为二分图                                         | 无奇圈+`DFS`或模拟染色                                       |



#### 最短（最长）路径

##### 病毒传播问题

- 病毒会感染周围，适合用 `BFS` 做，记录层数。（虽然作死用了 `DFS` ，但是根本做不出~）（`ID-1`）

##### 迷宫最短路径问题

- 走迷宫的题，直接 `BFS` 找最短路（`ID-2`，`ID-3`，`ID-4`）

##### Dijkstra

- 针对非负权路径，用**优先队列**优化（Go得手搓...，~~准备写个😭） （`ID-5`）

  ```go
  // 优先队列
  type node struct {
  	i, j, val int
  }
  
  type priorQueue struct {
  	nodes []node
  }
  
  func initPriorQueue() *priorQueue {
  	return &priorQueue{[]node{{-1, -1, -1}}}
  }
  
  func (p *priorQueue) Push(i, j, val int) {
  	idx := len(p.nodes)
  	p.nodes = append(p.nodes, node{i, j, val})
  	for ; p.nodes[idx].val < p.nodes[idx/2].val; idx /= 2 {
  		p.nodes[idx], p.nodes[idx/2] = p.nodes[idx/2], p.nodes[idx]
  	}
  }
  
  func (p *priorQueue) Pop() (i, j, val int) {
  	i, j, val = p.nodes[1].i, p.nodes[1].j, p.nodes[1].val
  	idx := len(p.nodes) - 1
  	p.nodes[1] = p.nodes[idx]
  	p.nodes = p.nodes[:idx]
  	l := idx
  	idx = 1
  	for (idx*2 < l && p.nodes[idx].val > p.nodes[idx*2].val) ||
  		(idx*2+1 < l && p.nodes[idx].val > p.nodes[idx*2+1].val) {
  		if (idx*2+1 >= l) || (p.nodes[idx*2].val < p.nodes[idx*2+1].val) {
  			p.nodes[idx], p.nodes[idx*2] = p.nodes[idx*2], p.nodes[idx]
  			idx *= 2
  		} else {
  			p.nodes[idx], p.nodes[idx*2+1] = p.nodes[idx*2+1], p.nodes[idx]
  			idx = idx*2 + 1
  		}
  	}
  	return
  }
  
  func (p *priorQueue) Empty() bool {
  	if len(p.nodes) <= 1 {
  		return true
  	}
  	return false
  }
  ```

  ```go
  func dijkstra(heights [][]int) int {
  	n, m := len(heights), len(heights[0])
  	vs := make([][]bool, n)
  	for i := 0; i < n; i++ {
  		vs[i] = make([]bool, m)
  	}
  	pq := initPriorQueue()
  	pq.Push(0, 0, 0)
  	for !pq.Empty() {
  		i, j, val := pq.Pop()
  		if vs[i][j] {
  			continue
  		}
  		if i == n-1 && j == m-1 {
  			return val
  		}
  		vs[i][j] = true
  		for _, surround := range surrounds {
  			if _i, _j := surround[0]+i, surround[1]+j; _i >= 0 && _i < n && _j >= 0 && _j < m && !vs[_i][_j] {
                  // 更新规则是max而不是更传统的add
  				pq.Push(_i, _j, max(val, heights[i][j]-heights[_i][_j]))
  			}
  		}
  	}
  	return -1
  }
  ```

  

##### 路径例题

| ID   | LeetCode 题号                                                | 描述                         | 思路                                                         |
| ---- | ------------------------------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| 1    | [994. Rotting Oranges](https://leetcode.cn/problems/rotting-oranges/) | 腐烂橘子向四周传播           | `BFS` + 广义层数记录                                         |
| 2    | [909. Snakes and Ladders](https://leetcode.cn/problems/snakes-and-ladders/) | 规则爬塔棋                   | `BFS` + 层数记录，注意题意，**我是傻逼**                     |
| 3    | [1091. Shortest Path in Binary Matrix](https://leetcode.cn/problems/shortest-path-in-binary-matrix/) | 迷宫最短路                   | `BFS` + 层数记录                                             |
| 4    | [1293. Shortest Path in a Grid with Obstacles Elimination](https://leetcode.cn/problems/shortest-path-in-a-grid-with-obstacles-elimination/) | 能够拆墙的迷宫游戏           | `BFS` + 记录是否走过时候记录走到这一步拆墙用过几次，次数高的就不再走了 |
| 5    | [1631. Path With Minimum Effort](https://leetcode.cn/problems/path-with-minimum-effort/) | 找到目的地的最小体力消耗路径 | `Dijkstra` + 不同的更新规则                                  |

并查集Union Find  准备模板
用于快速合并图的不同components （305. Number of Islands II）
用于快速判断两个nodes是不是连通
回溯法 Backtracking 本质就是想象成图，然后递归的DFS（有时可以剪枝)  526. Beautiful Arrangement, 22. Generate Parentheses

binarysearch+BFS： 用binary search 查找答案，然后在限制条件下做BFS。类似的用binary search 查找答案的思路见【7. 搜索和查询 中的binary search部分】
1102 Path With Maximum Minimum Value
778  Swim in Rising Water
1631 Path With Minimum Effort 



## 动态规划

- 动态规划（`Dynamic Programming`）

- **基本思路**，对于一个能用动态规划解决的问题，一般采用如下思路解决：

  1. 将原问题划分为若干 **阶段**，每个阶段对应若干个子问题，提取这些子问题的特征（称之为 **状态**）；
  2. 寻找每一个状态的可能 **决策**，或者说是各状态间的相互转移方式（用数学的语言描述就是 **状态转移方程**）。
  3. 按顺序求解每一个阶段的问题。

- 以 `ID-1` 为例，其状态转移方程为：$f(i+1)=\underset{0 \leq j \leq i}{max}f(j)$

  ```go
  func lengthOfLIS(nums []int) int {
  	ans := 0
  	dp := make([]int, len(nums))
  	for i := 0; i < len(nums); i++ {
  		dp[i] = 1
  		for j := i - 1; j >= 0; j-- {
  			if nums[i] > nums[j] {
  				dp[i] = max(dp[i], dp[j]+1)
  			}
  		}
  		ans = max(ans, dp[i])
  	}
  	return ans
  }
  ```

#### 记忆化搜索

- 记忆化搜索一般和 `DFS` 结合，传统的 `DFS` 时间复杂度不太可控，尤其是一次搜索后其他节点可以重复访问，而非迷宫问题那样可以一次访问后不需要再次访问，即 $O(n*m)$ 。因此，记忆化搜索出现了，**用空间换时间**。
- 如何写记忆化搜索：
  1. 把这道题的 `dp` 状态和方程写出来
  2. 根据它们写`DFS` 函数
  3. 添加记忆化数组

- 例如在 `ID-1` 中，我们可以记录搜过的状态（即每个点）：

  ```go
  func lengthOfLIS(nums []int) int {
  	mem := make([]int, len(nums))
  	var dfs func(i int) int
  	dfs = func(i int) (ret int) {
  		if mem[i] != 0 {
  			return mem[i]
  		}
  		ret++
  		for j := 0; j < i; j++ {
  			if nums[j] < nums[i] {
  				ret = max(ret, dfs(j)+1)
  			}
  		}
  		mem[i] = ret
  		return
  	}
  
  	ans := 0
  	for i := 0; i < len(nums); i++ {
  		if mem[i] == 0 {
  			ans = max(ans, dfs(i))
  		}
  	}
  	return ans
  }
  ```

#### 背包DP问题

- 背包问题是经典的 `dp` 题目，指一个容量有限的背包装最大价值的东西：

##### 0-1背包

> **0-1背包** 限制了物品有且只有一件，在背包最大质量为 $W$ 的情况下，能装物品价值最大，当然能有空位：

- 首先考虑遍历到第 $i$ 个物品时，有两种状态，取或者不取，那么对于重量 $0 \leq j \leq W$ ，可以通过之前的状态获得，则有递推方程：
  $$
  f[i][j]=max(f[i-1][j],f[i-1][j-w_i]+v_i)
  $$

- 显然这个方程只与 $i-1$ 的状态有关，就能通过滚动数组优化：
  $$
  f[j]=max(f[j],f[j-w_i]+v_i)
  $$

- 但是，问题来了，当我按照 $j: w_i \to W$ 的顺序更新时，会影响到后面的状态，即物品选取了多次，这与题意违背，所以，采用反向遍历的方式 $j: W \to w_i$ 。

  > **务必牢记并理解这个转移方程，因为大部分背包问题的转移方程都是在此基础上推导出来的。**

- **代码**：

  ```go
  for i := 0; i < len(w); i++ {
      for j := W; j >= w[i]; j-- {
          f[j] = max(f[j], f[j-w[i]]+v[i])
      }
  }
  ```



##### 完全背包

> **完全背包** **不限制**物品的件数，在背包最大质量为 $W$ 的情况下，能装物品价值最大，当然能有空位：

- 模仿 **0-1背包** 问题，我们给出的递推方程：
  $$
  f[i][j]=\underset{0 \leq k \leq +\infty}{max} \{ f[i-1][j-k \times w_i]\}+k \times v_i
  $$

- 然后优化这个方程，在替换为滚动数组的时候我们会发现，我们并不关心，顺序更新下（$j:0 \to W$），或者说可以利用之前更新的状态，而非担心被其污染；所以就有了以下方程：（跟 **0-1背包** 完全一样呢~）
  $$
  f[j]=max(f[j],f[j-w_i]+v_i) \qquad j:0 \to W
  $$

- **代码**：

  ```go
  for i := 0; i < len(w); i++ {
      for j := w[i]; j <= W; j++ {
          f[j] = max(f[j], f[j-w[i]]+v[i])
      }
  }
  ```



##### 多重背包

> **多重背包** **限制**物品的件数为 $k_i$，在背包最大质量为 $W$ 的情况下，能装物品价值最大，当然能有空位：

- 同样模仿 **0-1背包** ，我们可以朴素地写出递推方程：
  $$
  f[i][j]=\underset{0 \leq k \leq k_i}{max} \{f[i-1][j-k \times w_i]\}+k \times v_i
  $$

- 上述方程的时间复杂度为 $O(W \cdot \sum_{i=1}^{n}k_i)$ 。因此我们从 $\sum{k_i}$ 处下手。因为我们考虑了大量相同的选择方案，比如 $k$ 件物品其实是一样的，因此不需要组合数的复杂度来选择，所以，可以使用**二进制分组**的方法进行优化。

- 具体地说就是令 $A_{i,s}\; (s \in [0,\lfloor{log_2(k_i+1)}\rfloor-1])$ 分别表示由 $2^s$ 个单个物品“捆绑”而成的大物品。特殊地，若 $k_i+1$ **不是** $2$ 的整数次幂，则需要在最后添加一个由 $k_i-2^{\lfloor{log_2(k_i+1)}\rfloor-1}$ 个单个物品“捆绑”而成的大物品用于补足。时间复杂度降为：$O(W \cdot \sum_{i=1}^n log_2k_i)$



##### 二维费用背包

> 有 $n$ 个任务需要完成，完成第 $i$ 个任务需要花费 $t_i$ 分钟，产生 $c_i$ 元的开支。
>
> 现在有 $T$ 分钟时间，$C$ 元钱来处理这些任务，求最多能完成多少任务。

- 还是 **0-1背包** 的问题，我们直接从**优化后**的递推方程开始：
  $$
  f[i][j]=max(f[i][j],f[i-c[k]][j-t[k]]+1) \qquad 
  \begin{cases}
  	i:C \to c_i \\ 
  	j:T \to t_i \\
  \end{cases}
  $$
  



##### 分组背包

> 有 $n$ 件物品和一个大小为 $m$ 的背包，第 $i$ 个物品的价值为 $w_i$ ，体积为 $v_i$ 。同时，每个物品属于一个组，同组内最多只能选择一个物品，求背包能装载物品的最大总价值。

- 这种题怎么想呢？其实是从「在所有物品中选择一件」变成了「从当前组中选择一件」，于是就对每一组进行一次 0-1 背包就可以了。

- **代码**：

  ```go
  for k:=0; k<len(package); k++ {
      for i:=w; i>=0; i-- {
          for j:=0; j<len(package[k]); j++ {
              if i>=package[k][j].w {
                  dp[i]=max(dp[i],dp[i-package[i][j].w]+package[k][j].v)
              }
          }
      }
  } 
  ```

  > 这里要注意：**一定不能搞错循环顺序**，这样才能保证正确性。



#### 区间规划DP问题

-  **区间类动态规划**是线性动态规划的扩展，它在分阶段地划分问题时，与阶段中元素出现的顺序和由前一阶段的哪些元素合并而来有很大的关系。
- 令状态 $f[i][j]$ 表示将下标位置 $i$ 到 $j$ 的所有元素合并能获得的价值的最大值，那么 $f[i][j]=max\{f[i][k]+f[k+1][j]+cost\}$ ， $cost$ 为将这两组元素合并起来的代价。
- 区间 `dp` 有以下特点：
  - **合并**：即将两个或多个部分进行整合，当然也可以反过来；
  - **特征**：能将问题分解为能两两合并的形式；
  - **求解**：对整个问题设最优值，枚举合并点，将问题分解为左右两个部分，最后合并两个部分的最优值得到原问题的最优值。

> 在一个环上有 $n$ 个数 $a_1,a_2,\dots,a_n$，进行 $n-1$ 次合并操作，每次操作将相邻的两堆合并成一堆，能获得新的一堆中的石子数量的和的得分，你需要最大化你的得分。

- 首先考虑**链上而不是环上**的情况，状态转移方程很简单写出：
  $$
  f[i][j]=\underset{i\leq k \leq j-1}{max}\{f[i][k]+f[k+1][j]\}
  $$

- 那么处理环上呢，我们可以复制整个链，变成 $2 \times n$ 个，其中第 $i$ 堆与第 $n+i$ 堆相同，用动态规划求解后，取 $f[1][n],f[2][n+2], \dots ,f[n-1][2n-2]$中的最优值，为最后的答案。



#### DAG有向无环图DP问题

1. **建立DAG**，每一个节点表示一种姿态的方块 $(h,(a,b))$ ，其中 $(a.b)$ 是无序的，那么我们能建立一个DAG了。我们将以两种方块为例：$(31,41,59)$ 和 $(33,83,27)$

   ![img](images/dag-babylon.png)

   图中黄色虚线框表示的是重复计算部分，可以采用**记忆化搜索**的方法来避免重复计算。

2. 下面我们开始考虑转移方程，设 $d(i,r)$ 表示第 $i$ 块砖块在最上面，且采用第 $r$ 种堆叠方式的最大高度。那么有如下的转移方程：
   $$
   d(i,r)=max\{d(j,r')+h\}
   $$



#### 树形DP问题

- 树形 DP，即在树上进行的 DP。由于树固有的递归性质，树形 DP 一般都是递归进行的。

> 某大学有 $n$ 个职员，编号为 $1 \sim N$ 。他们之间有从属关系，也就是说他们的关系就像一棵**以校长为根**的树，父结点就是子结点的直接上司。现在有个周年庆宴会，宴会每邀请来一个职员都会增加一定的快乐指数 ，但是呢，如果某个职员的直接上司来参加舞会了，那么这个职员就无论如何也不肯来参加舞会了。所以，请你编程计算，邀请哪些职员可以使快乐指数最大，求最大的快乐指数。

1. 我们设 $f(i,0/1)$ 代表以 $i$ 为根的子树的最优解（第二维的值为 $0$ 代表 $i$ 不参加舞会的情况，$1$ 代表 $i$ 参加舞会的情况）。对于每个状态，都存在两种决策（其中下面的 $x$ 都是 $i$ 的儿子）：
   $$
   \begin{align}
   f[i][0] &= \sum \max\{f[x][1],f[x][0]\} 	\\
   f[i][1] &= \sum f[x][0] + a_i				\\
   \end{align}
   $$

2. 通过 `DFS` ，在返回上一层时更新当前节点的最优解。

##### 树上背包问题

> 现在有 $n$ 门课程，第 $i$ 门课程的学分为 $a_i$，每门课程有零门或一门先修课，有先修课的课程需要先学完其先修课，才能学习该课程。一位学生要学习 $m$ 门课程，求其能获得的最多学分数。

1. 每门课最多只有一门先修课的特点，与有根树中一个点最多只有一个父亲结点的特点类似，因此可以想到根据这一性质建树。

2. 如果所有课程组成了一个森林的结构，为了方便起见，我们可以新增一门 学分的课程（设这个课程的编号为 $-1$），作为所有无先修课课程的先修课，这样我们就将森林变成了一棵以 $-1$ 号课程为根的树。

3. 接着模仿基础的树形DP和背包问题，提出递归过程，设 $f[u][i][j]$ 表示点 $u$ 为根的子树中，已经遍历了 $u$ 的前 $i$ 棵子树，选了 $j$ 门课程的最大学分，记点 $u$ 的儿子个数为 $s_u$，以 $x$ 为根的子树大小为 $size_x$，可以写出下面的状态转移方程：
   $$
   f[u][i][j]=\underset{v,k \leq j,k \leq size_v}{\max}\{f[u][i-1][j-k]+f[v][s_v][k]\}
   $$

4. **参考代码**：

   ```c++
   #include <algorithm>
   #include <cstdio>
   #include <vector>
   using namespace std;
   int f[305][305], s[305], n, m;
   vector<int> e[305];
   
   int dfs(int u) {
     int p = 1;
     f[u][1] = s[u];
     for (auto v : e[u]) {
       int siz = dfs(v);
       // 注意下面两重循环的上界和下界
       // 只考虑已经合并过的子树，以及选的课程数超过 m+1 的状态没有意义
       for (int i = min(p, m + 1); i; i--)
         for (int j = 1; j <= siz && i + j <= m + 1; j++)
           f[u][i + j] = max(f[u][i + j], f[u][i] + f[v][j]);  // 转移方程
       p += siz;
     }
     return p;
   }
   
   int main() {
     scanf("%d%d", &n, &m);
     for (int i = 1; i <= n; i++) {
       int k;
       scanf("%d%d", &k, &s[i]);
       e[k].push_back(i);
     }
     dfs(0);
     printf("%d", f[0][m + 1]);
     return 0;
   }
   ```

5. （有空自己写个 Go 的...）



##### 换根DP

- 树形 DP 中的换根 DP 问题又被称为二次扫描，通常不会指定根结点，并且根结点的变化会对一些值，例如子结点深度和、点权和等产生影响。

- 通常需要两次 `DFS`，第一次 `DFS` 预处理诸如深度，点权和之类的信息，在第二次 `DFS` 开始运行换根动态规划。

> 给定一个 $n$ 个点的树，请求出一个结点，使得以这个结点为根时，所有结点的深度之和最大。

1. 不妨令 $u$ 为当前结点， 为当前结点的子结点。首先需要用 $s_i$ 来表示以 $i$ 为根的子树中的结点个数，并且有 $s_u=1+\sum s_v$ 。显然需要一次 `DFS` 来计算所有的 $s_i$ ，这次的 `DFS` 就是预处理，我们得到了以某个结点为根时其子树中的结点总数。

2. 考虑状态转移，这里就是体现＂换根＂的地方了。令 $f_u$ 为以 $u$ 为根时，所有结点的深度之和。

3. $f_v \leftarrow f_u$ 可以体现换根，即以 $u$ 为根转移到以 $v$ 为根。显然在换根的转移过程中，以 $u$ 为根或以 $v$ 为根会导致其子树中的结点的深度产生改变。具体表现为：

   - 所有在 $v$ 的子树上的结点深度都减少了 $1$，那么总深度和就减少了 $s_v$ ；

   - 所有不在 $v$ 的子树上的结点深度都增加了 $1$，那么总深度和就增加了 $n-s_v$ ；

3. 根据这两个条件就可以推出状态转移方程 $f_v=f_u-s_v+n-s_v=f_u+n-2 \times s_v$ 。

4. 于是在第二次 `DFS` 遍历整棵树并状态转移 $f_v=f_u+n-2 \times s_v$ ，那么就能求出以每个结点为根时的深度和了，最后只需要遍历一次所有根结点深度和就可以求出答案。



##### 状压DP

- 状压 DP 是动态规划的一种，通过将状态压缩为整数来达到优化转移的目的。

> 在 $N \times N$ 的棋盘里面放 $K$ 个国王（$1 \leq N \leq 9, 1 \leq K \leq N \times N$），使他们互不攻击，共有多少种摆放方案。国王能攻击到它上下左右，以及左上左下右上右下八个方向上附近的各一个格子，共 $8$ 个格子。

1. 设 $f[i][j][l]$ 表示前 $i$ 行，第 $i$ 行的状态为 $j$ ，且棋盘上已经放置 $l$ 个国王时的合法方案数。$x$ 为上一行某个状态，$sta(j)$ 表示状态 $j$里的国王个数，那么**状态转移方程**：
   $$
   f[i][j][l]=\sum f[i-1][x][l-sta(j)]
   $$





#### 动态规划例题

| ID   | LeetCode 题号                                                | 描述           | 思路                                                         |
| ---- | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| 1    | [300. Longest Increasing Subsequence](https://leetcode.cn/problems/longest-increasing-subsequence/) | 最长递增子序列 | 一维 `dp`，$f[i+1]=\underset{0 \leq j \leq i}{max}\{f[j]\}$  |
| 2    | [1143. Longest Common Subsequence](https://leetcode.cn/problems/longest-common-subsequence/) | 最长公共子序列 | 二维 `dp`，$f[i+1][j+1] = \begin{cases} f[i][j] & str1[i]=str2[j] \\max(f[i+1][j],f[i][j+1]) & str1[i] \ne str2[j] \end{cases}$ |
|      |                                                              |                |                                                              |

有状态转化方程，可以把大问题转化为几个小问题，或者可以按某种顺序依次解决问题。（用图的思想，data是node, operation是edge）
常见思路
用dp代表关于arr[0:i]的subproblem  (只到i 或者 从i开始的subproblem）
用dp[j] 代表关于arr[i:j+1]的subproblem （或者是关于两个数组的 arr[0:i] 和 arr2[0:j]的subproblem, 或者关于两个变量i,j的subproblem）
经典DP题目.
 (2D version: 354. Russian Doll Envelopes)

Longest Substring Without Repeating Characters
字符串操作： 72. Edit Distance， 44. Wildcard Matching, 10. Regular Expression Matching
Palindrome problems: 647. Palindromic Substrings, 5. Longest Palindromic Substring
Prefix sum/max/min 相关： 42. Trapping Rain Water, 1423. Maximum Points You Can Obtain from Cards， Range Sum Query - Immutable, 304. Range Sum Query 2D - Immutable
Word Break 系列
硬币零钱系列 Coin Change
买股票系列 Best Time to Buy and Sell Stock
跳跃游戏系列 Jump games
抢劫系列 House Robber
石头游戏系列（Alice & Bob) Stone Game
Unique Paths 系列
688 Knight Probability in Chessboard
摘樱桃 Pick cherry
174  Dungeon Game
1277 Count Square Submatrices with All Ones
加油站问题 871. Minimum Number of Refueling Stops



## 树 Tree 

树的遍历
DFS (binary tree: in-order, pre-order,  post-order)
BFS: 314. Binary Tree Vertical Order Traversal， 199. Binary Tree Right Side View
递归大法 (大部分树的题都能递归，大的问题(root)，等于先解决几个子问题（subtree), 然后合并）:
124 Binary Tree Maximum Path Sum，
366 Find Leaves of Binary Tree
Lowest Common Ancestor系列
Binary Search Tree 判断和快速查找元素 98. Validate Binary Search Tree
树的编码和解码
297 Serialize and Deserialize Binary Tree
428 Serialize and Deserialize N-ary Tree
把树变成图： 863. All Nodes Distance K in Binary Tree

1. 
2. 堆 Heap, 栈 Stack, 队 Queue
  栈 Stack. 1point 3 acres
  常规题
  946 Validate Stack Sequences
  Asteroid Collision
  括号题
  Valid Parentheses, Remove Invalid Parentheses
  Basic Calculator 系列
  Nested List Iterator 系列
  Decode String， Number of Atoms. .и
  单调栈
  Next Greater Element 系列
  402 Remove K Digits
  853 Car Fleet
  739 Daily Temperatures
  堆 Heap
  Top k： 215. Kth Largest Element in an Array， 347. Top K Frequent Elements
  中位数： double heap 295. Find Median from Data Stream
  另外一道经典中位数题目 4. Median of Two Sorted Arrays. 1point3acres
  会议室问题 253. Meeting Rooms II
  CPU分配 模板: LC 1834. Single-Threaded CPU, LC 1882. Process Tasks Using Servers.1point3acres
  队 Queue, Deque
  BFS related
  239 Sliding Window Maximum  ---> 2D sliding window maximum ( 转化成两次1D的问题）
  Moving Average from Data Stream
3. 链表 LinkedList
  Fast and Slow pointer  (detect cycle, get middle,  get kth element)
  141 Linked List Cycle
  19 Remove Nth Node From End of List
  Reverse Linked List (trick: dummy head)  206 Reverse Linked List, 25 Reverse Nodes in k-Group
  LRU cache
  Deep copy  (138 Copy List with Random Pointer)
  Merge LinkedList  (2. Add Two Numbers)
4. 排序 Sort
  Merge sort
  非常规高频题 315 Count of Smaller Numbers After Self  --> (google题： 一堆点, 对每个点(x,y)数【严格大 (x,y)<(u,v)】的点的个数. 思路：先排序，x增序，y减序，然后把y单独拿出来看，对每个点数右边有多少大的元素，变成问题315 with bigger numbers after self)
  Quick Sort --> QuickSelect O(n) time on average) 973. K Closest Points to Origin
  Bucket Sort O(n) 通常是整体数据量可能很大，但是unique元素有限
  Cycle Sort O(n) 通常是用于把0到n-1在array中排序 （不断交换的想法）
  Python built-in sort
  OrderedDict (linked list + hash) --> 自己实现： 用 hashtable 存double linkedlist 的 node
  sorted containers (sorted list, sorted dict, sorted set)
5. 搜索和查询 Search and Query
  hash (python: dictionary, set):
  O(1)查找，
  记录unique element的frequency
  binary search  左开右闭模板
  data是有顺序的，每次可以缩小搜索范围。 经典题：
  33 Search in Rotated Sorted Array，
  153 Find Minimum in Rotated Sorted Array，
  162 Find Peak Element. check 1point3acres for more.
  解的范围是一个区间可以二分搜索
  Binary search + greedy: 1231 Divide Chocolate, 1011 Capacity To Ship Packages Within D Days, 410. Split Array Largest Sum
  378 Kth Smallest Element in a Sorted Matrix
  经典题 Search a 2D Matrix 系列
  字典树 Trie 模板 （单词相关的查找）： 642. Design Search Autocomplete System， 472. Concatenated Words， 212. Word Search II
  Range Query (Segment Tree 模板)  307. Range Sum Query - Mutable. check 1point3acres for more.
6. 数组和字符串相关 （array & string）
  括号相关题 （另外见【栈】）921. Minimum Add to Make Parentheses Valid， 1249. Minimum Remove to Make Valid Parentheses
  排列（组合） Permutation
  区间题 Intervals [left, right, val]
  按左右端点排序的思想 252. Meeting Rooms
  插入，合并，删除区间： 56. Merge Intervals， 57. Insert Interval， 1272. Remove Interval， 435. Non-overlapping Intervals
  安排会议/任务  253. Meeting Rooms II， 1235 Maximum Profit in Job Scheduling， 2054 Two Best Non-Overlapping Events
  区间更新 1094. Car Pooling. 1point 3acres
  常规双指针
  15 3Sum，
  75 Sort Colors（Dutch national flag problem 经典题
  1229 Meeting Scheduler，
  680 Valid Palindrome II，
  408 Valid Word Abbreviation
  滑动窗口 Sliding window 模板. 1point 3 acres
  3 Longest Substring Without Repeating Characters
  76 Minimum Window Substring
  1004 Max Consecutive Ones III
  209  Minimum Size Subarray Sum
  1438 Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit
  Subsequence.
  Greedy drop idea: 392 Is Subsequence， 792 Number of Matching Subsequences
  经典题 727. Minimum Window Subsequence
  940 Distinct Subsequences II
  Subarray、substring （连续的）. From 1point 3acres bbs
  Rolling hash （Rabin-Karp） 1062. Longest Repeating Substring， 1044. Longest Duplicate Substring
  排列组合
  46 Permutations, 31. Next Permutation
  77 Combinations
  Subset
  78 Subsets
  368 Largest Divisible Subset
  数据流相关的问题
  top k 问题 --> heap; buck sort -->  distributed system:
  1146 Snapshot Array  (打version tag + binary search）
  359 Logger Rate Limiter
  LRU, LFU
  Median 295. Find Median from Data Stream
  Iterator  284. Peeking Iterator, 900. RLE Iterator. 1point3acres.com
  Bitmask 用bit来表示状态 847. Shortest Path Visiting All Nodes
  补充内容 (2022-01-20 13:20 +8:00):
  最近转码上岸了，总结了一下我做过的一些有代表的力扣题，回馈地里，希望对大家有帮助！
  另外还有一篇【转码找工作的资料总结】还在审核中
  链接： https://www.1point3acres.com/bbs/thread-840857-1-1.html
  .1point3acres
  补充内容 (2022-01-21 02:13 +8:00):. From 1point 3acres bbs
  转码资料的帖子通过审核啦！
  Dynamic programming 那儿格式出了点问题，应该是 dp 和 dp[j] 这两种最常见的方式 . .и
  补充内容 (2022-01-21 03:51 +8:00):
  还是有问题。。。。
  是 dp_i   和 dp_i,j
