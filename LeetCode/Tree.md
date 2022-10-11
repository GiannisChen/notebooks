## 树（Tree ）

- 是一个一个一个无环图罢了（悲~

#### 树的遍历

- 树的遍历可以参考[图的遍历](#图的遍历)，其他稍微有点特殊的另外给出：

##### DFS（先序，中序，后序）

- **先序**（`pre-order`）[144. Binary Tree Preorder Traversal](https://leetcode.cn/problems/binary-tree-preorder-traversal/)

  ```go
  func preOrder(node *TreeNode) {
      if node != nil {
          ...
          preOrder(node.Left)
          preOrder(node.Right)
      }
  }
  ```

  ```go
  func preOrder(node *TreeNode) {
      stack := make([]*TreeNode, 0)
      ptr := root
      for ptr != nil || len(stack) != 0 {
          if ptr != nil {
              ...
              stack = append(stack, ptr)
              ptr = ptr.Left
          }  else {
              ptr = stack[len(stack)-1].Right
              stack = stack[:len(stack)-1]
          }
      }
  }
  ```

- **中序**（`in-order`）[94. Binary Tree Inorder Traversal](https://leetcode.cn/problems/binary-tree-inorder-traversal/)

  ```go
  func inOrder(node *TreeNode) {
      if node != nil {
          preOrder(node.Left)
          ...
          preOrder(node.Right)
      }
  }
  ```

  ```go
  func inOrder(node *TreeNode) {
      stack := make([]*TreeNode, 0)
      ptr := root
      for ptr != nil || len(stack) != 0 {
          if ptr != nil {
              stack = append(stack, ptr)
              ptr = ptr.Left
          }  else {
              ptr = stack[len(stack)-1].Right
              ...
              stack = stack[:len(stack)-1]
          }
      }
  }
  ```

- **后序**（`post-order`）[145. Binary Tree Postorder Traversal](https://leetcode.cn/problems/binary-tree-postorder-traversal/)

  ```go
  func preOrder(node *TreeNode) {
      if node != nil {
          ...
          preOrder(node.Left)
          preOrder(node.Right)
      }
  }
  ```

  ```go
  func inOrder(node *TreeNode) {
      stack := make([]*TreeNode, 0)
      visited := make([]bool, 0)
      res := make([]*TreeNode, 0)
      ptr := root
      stack = append(stack, ptr)
      visited = append(visited, false)
      
      for len(stack) != 0 {
          ptr, v = stack[len(stack)-1], visited[len(stack)-1]
          stack, visited = stack[:len(stack)-1], visited[:len(stack)-1]
          if v {
              res = append(res, ptr)
          }  else {
              stack = append(stack, ptr)
      		visited = append(visited, true)
              if ptr.Right != nil {
                  stack = append(stack, ptr.Right)
  	    		visited = append(visited, false)
              }
              if ptr.Left != nil {
                  stack = append(stack, ptr.Left)
  	    		visited = append(visited, false)
              }
          }
      }
  }
  ```

- 树的遍历题很多都是可以通过**递归**来解决，经典的是 `DFS` 。

#### 树的遍历例题

| ID   | LeetCode 题号                                                | 描述                        | 思路                                                         |
| ---- | ------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| 1    | [199. Binary Tree Right Side View](https://leetcode.cn/problems/binary-tree-right-side-view/) | 树的右视图                  | 层序遍历 `BFS`，每次添加层末尾元素                           |
| 2    | [124. Binary Tree Maximum Path Sum](https://leetcode.cn/problems/binary-tree-maximum-path-sum/) | 无向树中权最大的路径        | `DFS` 更新和返回值分开，**更新**当前节点 + 左子树递归 + 右子树递归，**返回**当前节点 + 子树递归最大值 |
| 3    | [98. Validate Binary Search Tree](https://leetcode.cn/problems/validate-binary-search-tree/) | 验证是否是二叉搜索树        | 要注意整棵子树和根节点的大小关系                             |
| 4    | [297. Serialize and Deserialize Binary Tree](https://leetcode.cn/problems/serialize-and-deserialize-binary-tree/) | 二叉树的编码和解码          | `DFS` + 按序遍历                                             |
| 5    | [863. All Nodes Distance K in Binary Tree](https://leetcode.cn/problems/all-nodes-distance-k-in-binary-tree/) | 找距离某节点长度为k的所有点 | `DFS` ，通过记录父节点统一 `DFS` 的过程，通过哈希表排除重复节点 |

| 题                                        | 题                             | 题                         | 题                                       |
| ----------------------------------------- | ------------------------------ | -------------------------- | ---------------------------------------- |
| 314. Binary Tree Vertical Order Traversal | 366 Find Leaves of Binary Tree | Lowest Common Ancestor系列 | 428 Serialize and Deserialize N-ary Tree |
|                                           |                                |                            |                                          |
|                                           |                                |                            |                                          |



#### 线段树（Segment Tree）

**参考**

1. https://leetcode.cn/problems/range-sum-query-mutable/solution/by-lfool-v3x9/



- 一颗线段树长如下图所示，其本质是用于范围统计（$\sum^{r}_{i=l}$）的一个树状结构，采用二分思想；其叶子节点存放着单点信息，而其他非叶节点则存放着范围信息。这就意味着所有范围操作都能**递归**进行，代码简单（但不直白）。

<img src="images/segment-tree-00.svg" alt="img" style="zoom:67%;" />

- 常见的范围统计操作无外乎：$sum$ ， $max$ ，$min$ ，$avg$ 此类。



##### 已有数组→线段树

- 当给定一个数组时，我们需要在这个基础上“长”出一颗线段树：（**自底向上**）

<img src="images/segment-tree-01.svg">

- 这种线段树的索引构思很巧妙，我们不难发现，**最底层**的节点**二进制**最低位为 $1$，**倒数第二层**的最低位为 $10$，以此类推，我们就能得到递推公式：`i = i + (i & -i)`。以题[307. Range Sum Query - Mutable](https://leetcode.cn/problems/range-sum-query-mutable/)为例：

- **数据结构**

  ```go
  type NumArray struct {
  	nums []int
  	tree []int
  	n    int
  }
  ```

- **添加**和**更新**

  ```go
  func (na *NumArray) add(index int, val int) {
  	for i := index + 1; i <= na.n; i += lowBit(i) {
  		na.tree[i] += val
  	}
  }
  
  func (na *NumArray) update(index int, val int) {
  	na.add(index, val-na.nums[index])
  	na.nums[index] = val
  }
  ```

- **范围查询**（这一步区别于标准线段树，他每个节点的和等于从 $1$ 号结点开始到当前节点的数据和，所以这里的 $left$ 其实是 $left+1-1$）

  ```go
  func (na *NumArray) query(i int) (res int) {
  	for ; i > 0; i -= lowBit(i) {
  		res += na.tree[i]
  	}
  	return
  }
  
  func (na *NumArray) sumRange(left int, right int) int {
  	return na.query(right+1) - na.query(left)
  }
  ```



##### 动态开线段树

- 堆式储存的情况下，需要给线段树开 $4n$ 大小的数组（这个数据是 $log_2$ 累加算出来的）。为了节省空间，我们可以不一次性建好树，而是：

  1. 在最初只建立一个根结点代表整个区间；
  2. 当我们需要访问某个子区间时，才建立代表这个区间的子结点；这样我们不再使用 $2p$ 和 $2p+1$ 代表 $p$ 结点的儿子，而是用 $ls$ 和 $rs$ 记录儿子的编号。总之，动态开点线段树的核心思想就是：**结点只有在有需要的时候才被创建**。

  > 单次操作的时间复杂度是不变的，为 $O(log\,n)$ 。由于每次操作都有可能创建并访问全新的一系列结点，因此 $m$ 次单点操作后结点的数量规模是 $O(m\,log\,n)$。最多也只需要 $2n-1$ 个结点，没有浪费。

- **懒惰标记优化**：更新时线段树讲究向下创建节点，但是现在我不了，对于已有分段我不创建，而是使用**懒惰标记**表示一次或者多次更新，而放到后面再做操作。

- **`map` 版本**：（以 [729. My Calendar I](https://leetcode.cn/problems/my-calendar-i/) 为题）

  ```go
  type MyCalendar struct {
  	tree, lazy map[int]bool
  }
  
  func Constructor() MyCalendar {
  	return MyCalendar{
  		tree: map[int]bool{},
  		lazy: map[int]bool{},
  	}
  }
  
  func (c *MyCalendar) query(start, end int, left, right int, idx int) bool {
  	if right < start || left > end {
  		return false
  	}
  	if c.lazy[idx] {
  		return true
  	}
  	if start <= left && right <= end {
  		return c.tree[idx]
  	}
  	mid := (left + right) >> 1
  	return c.query(start, end, left, mid, idx*2) ||
  		c.query(start, end, mid+1, right, idx*2+1)
  }
  
  func (c *MyCalendar) update(start, end int, left, right int, idx int) {
  	if right < start || left > end {
  		return
  	}
  	if start <= left && right <= end {
  		c.tree[idx] = true
  		c.lazy[idx] = true
  	} else {
  		mid := (left + right) >> 1
  		c.update(start, end, left, mid, idx*2)
  		c.update(start, end, mid+1, right, idx*2+1)
  		c.tree[idx] = true
  		if c.lazy[2*idx] && c.lazy[2*idx+1] {
  			c.lazy[idx] = true
  		}
  	}
  }
  
  func (c *MyCalendar) Book(start int, end int) bool {
  	if c.query(start, end-1, 0, 1e9, 1) {
  		return false
  	}
  	c.update(start, end-1, 0, 1e9, 1)
  	return true
  }
  ```

- **`tree` 版本**：

  ```go
  type node struct {
  	l, r        int
  	left, right *node
  	lazy        bool
  	val         bool
  }
  
  type MyCalendar struct {
  	root *node
  }
  
  func Constructor() MyCalendar {
  	return MyCalendar{&node{l: 0, r: 1e9, lazy: false, val: false}}
  }
  
  func query(cur *node, start, end int) bool {
  	if start > cur.r || end < cur.l {
  		return false
  	}
  	if start <= cur.l && cur.r <= end {
  		return cur.val
  	}
  
  	lazyCreate(cur)
  	return query(cur.left, start, end) || query(cur.right, start, end)
  }
  
  func update(cur *node, start, end int) {
  	if start > cur.r || end < cur.l {
  		return
  	}
  	if start <= cur.l && cur.r <= end {
  		cur.lazy = true
  		cur.val = true
  		return
  	}
  
  	lazyCreate(cur)
  	update(cur.left, start, end)
  	update(cur.right, start, end)
  	cur.val = cur.left.val || cur.right.val
  }
  
  func lazyCreate(cur *node) {
  	mid := cur.l + (cur.r-cur.l)>>1
  	if cur.left == nil {
  		cur.left = &node{l: cur.l, r: mid}
  	}
  	if cur.right == nil {
  		cur.right = &node{l: mid + 1, r: cur.r}
  	}
  	if !cur.lazy {
  		return
  	}
  	cur.left.lazy = cur.lazy
  	cur.right.lazy = cur.lazy
  	cur.left.val = cur.left.val || cur.lazy
  	cur.right.val = cur.right.val || cur.lazy
  	cur.lazy = false
  }
  
  func (c *MyCalendar) Book(start int, end int) bool {
  	if query(c.root, start, end-1) {
  		return false
  	}
  	update(c.root, start, end-1)
  	return true
  }
  ```

  

#### 线段树例题

| ID   | LeetCode 题号                                                | 描述                           | 思路                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------ | ------------------------------------------------------------ |
| 1    | [307. Range Sum Query - Mutable](https://leetcode.cn/problems/range-sum-query-mutable/) | 区域检索和更新                 | 从给定的数组生成线段树                                       |
| 2    | [729. My Calendar I](https://leetcode.cn/problems/my-calendar-i/) | 日程预定列表                   | 线段树 + 懒惰标记                                            |
| 3    | [731. My Calendar II](https://leetcode.cn/problems/my-calendar-ii/) | 日程预定列表（可重复预定）     | 线段树 + 懒惰标记，注意更新方式                              |
| 4    | [732. My Calendar III](https://leetcode.cn/problems/my-calendar-iii/) | 日程预定列表（计算最大重复度） | `LeetCode`不会解释例子可以不解释嗷，万能模板：线段树 + 懒惰标记 |
| 5    | [2407. Longest Increasing Subsequence II](https://leetcode.cn/problems/longest-increasing-subsequence-ii/) | 最长递增子序列，但是有间隔限制 | 动态开树的线段树优化找的过程，原本还有一维DP的，结果发现不需要，**我是傻逼** |
| 6    | [715. Range Module](https://leetcode.cn/problems/range-module/) | 范围查询                       | 线段树                                                       |

| 题              | 题              | 题                  |
| --------------- | --------------- | ------------------- |
| 715. Range 模块 | 699. 掉落的方块 | 933. 最近的请求次数 |