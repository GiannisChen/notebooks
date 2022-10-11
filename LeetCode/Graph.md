## 图（Graph）

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
