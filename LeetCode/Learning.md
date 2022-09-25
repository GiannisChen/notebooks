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

| ID   | LeetCode 题号                                                | 描述                                  |
| ---- | ------------------------------------------------------------ | ------------------------------------- |
| 1    | [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/) | 相互不连接的岛屿的个数                |
| 2    | [690. Employee Importance](https://leetcode.cn/problems/employee-importance/) | 递归返回自身和下属重要性之和          |
| 3    | [79. Word Search](https://leetcode.cn/problems/word-search/) | 在字母表里找可能出现的字符串          |
| 4    | [1254. Number of Closed Islands](https://leetcode.cn/problems/number-of-closed-islands/) | 四周都被水包围的岛屿个数              |
| 5    | [1905. Count Sub Islands](https://leetcode.cn/problems/count-sub-islands/) | 第二张图的岛屿必须建在第一个岛屿上    |
| 6    | [417. Pacific Atlantic Water Flow](https://leetcode.cn/problems/pacific-atlantic-water-flow/) | 不同海拔的海岛是否可达上下两侧        |
| 7    | [127. Word Ladder](https://leetcode.cn/problems/word-ladder/) | 单词接龙，每次只能变化一个字符 `hard` |
| 8    | [695. Max Area of Island](https://leetcode.cn/problems/max-area-of-island/) | 返回面积最大的岛的面积                |
| 9    | [1345. Jump Game IV](https://leetcode.cn/problems/jump-game-iv/) | 按规则跳跃                            |



#### 拓扑排序

- `Topological Sort`

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
   
     ![bridge-edge-graph](../../../Drawios/bridge-edge-graph.svg)
   
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

图二分染色 (785. Is Graph Bipartite?)



##### 例题

| ID   | LeetCode 题号                                                | 描述                                                 |
| ---- | ------------------------------------------------------------ | ---------------------------------------------------- |
| 1    | [210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/) | 经典图论中先后顺序问题                               |
| 2    | [269. Alien Dictionary](https://leetcode.cn/problems/alien-dictionary/) | 给定一个按字典升序排列的字符串数组，判定字符先后顺序 |
| 3    | [207. Course Schedule](https://leetcode.cn/problems/course-schedule/) | 图论中是否成环的问题                                 |
| 4    | [1192. Critical Connections in a Network](https://leetcode.cn/problems/critical-connections-in-a-network/) | 图中是否有桥                                         |



最短（最长）路径
经典BFS题 994. Rotting Oranges, 909. Snakes and Ladders, 1091. Shortest Path in Binary Matrix, 1293. Shortest Path in a Grid with Obstacles Elimination
Dijkstra （用heap 写，准备模板） （1631. Path With Minimum Effort， 1066. Campus Bikes II）.
并查集Union Find  准备模板
用于快速合并图的不同components （305. Number of Islands II）
用于快速判断两个nodes是不是连通
回溯法 Backtracking 本质就是想象成图，然后递归的DFS（有时可以剪枝)  526. Beautiful Arrangement, 22. Generate Parentheses

binarysearch+BFS： 用binary search 查找答案，然后在限制条件下做BFS。类似的用binary search 查找答案的思路见【7. 搜索和查询 中的binary search部分】
1102 Path With Maximum Minimum Value
778  Swim in Rising Water
1631 Path With Minimum Effort 

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

1. 动态规划 Dynamic Programming
  有状态转化方程，可以把大问题转化为几个小问题，或者可以按某种顺序依次解决问题。（用图的思想，data是node, operation是edge）
  常见思路
  用dp代表关于arr[0:i]的subproblem  (只到i 或者 从i开始的subproblem）
  用dp[j] 代表关于arr[i:j+1]的subproblem （或者是关于两个数组的 arr[0:i] 和 arr2[0:j]的subproblem, 或者关于两个变量i,j的subproblem）
  经典DP题目. check 1point3acres for more.
  LIS: 300. Longest Increasing Subsequence O(nlogn)  (2D version: 354. Russian Doll Envelopes)
  LCS: 1143. Longest Common Subsequence
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



# 思路

## 图

- 见[图的遍历](#图的遍历例题)

| ID   | LeetCode 题号                                                | 思路                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | [200. Number of Islands](https://leetcode.cn/problems/number-of-islands/) | 直接 `DFS` 或者 `BFS` ，标记访问过的位置                     |
| 2    | [690. Employee Importance](https://leetcode.cn/problems/employee-importance/) | 广义图论，`DFS` 或 `BFS` ，甚至不用建图                      |
| 3    | [79. Word Search](https://leetcode.cn/problems/word-search/) | `DFS`，由于字母表最大容量较小，采用位运算来记录是否访问过，而不是矩阵 |
| 4    | [1254. Number of Closed Islands](https://leetcode.cn/problems/number-of-closed-islands/) | `ID-1` 的变种，只需把边界一圈先遍历掉，但不计数即可          |
| 5    | [1905. Count Sub Islands](https://leetcode.cn/problems/count-sub-islands/) | 需要遍历完，不然会产生额外的岛，其他没啥和 `ID-4` 一样       |
| 6    | [417. Pacific Atlantic Water Flow](https://leetcode.cn/problems/pacific-atlantic-water-flow/) | `DFS` 反向操作，从边上向中心传播                             |
| 7    | [127. Word Ladder](https://leetcode.cn/problems/word-ladder/) | `BFS` ，邻接表，用双向队列控制扇出                           |
| 8    | [695. Max Area of Island](https://leetcode.cn/problems/max-area-of-island/) | `DFS` 多一个返回值，很直白                                   |
| 9    | [1345. Jump Game IV](https://leetcode.cn/problems/jump-game-iv/) | `DFS`，我用了双队列优化，通过删邻接表来控制队列规模          |

- 见[拓扑排序](#拓扑排序)

| ID   | LeetCode 题号                                                | 思路                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | [210. Course Schedule II](https://leetcode.cn/problems/course-schedule-ii/) | `BFS` `DFS` 都可以，注意谁先谁后就行，可以通过去入度时候进队列来判断是否为入度零 |
| 2    | [269. Alien Dictionary](https://leetcode.cn/problems/alien-dictionary/) | 按照规则构建对应先后顺序表就行，小细节                       |
| 3    | [207. Course Schedule](https://leetcode.cn/problems/course-schedule/) | `DFS` + 状态数组                                             |
| 4    | [1192. Critical Connections in a Network](https://leetcode.cn/problems/critical-connections-in-a-network/) | 判断是否有桥，板子 `tarjan` 算法                             |