## 树状DP

- https://leetcode.cn/problems/count-subtrees-with-max-distance-between-cities/solution/tu-jie-on3-mei-ju-zhi-jing-duan-dian-che-am2n/

树状DP通常和**DFS**，状态压缩，回溯绑在一起，顾名思义，就是在非线性的数据结构上完成DP的过程。



#### 餐前水果

- [543. Diameter of Binary Tree](https://leetcode.cn/problems/diameter-of-binary-tree/)

二进制树的直径，被定义为二进制树中最长的通路。那么我们就想着如何分类：对于一个节点，最长的通路有两种情况，一种是以自身为根，向左右子树扩展，一种是穿过自己，向更高层的祖先节点发展。这两种不同的类别，可以使用**DFS**来解决：（数学归纳法）

- 我们每次都将更新`ans`，用的是`dfs(node.Left)`+`dfs(node.Right)`；
- 同时，我们通过`dfs`的返回值解决第二种情况，返回`dfs(node.Left)`和`dfs(node.Right)`的最大值并在路径长度上加上当前节点；

##### 代码

```go
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func diameterOfBinaryTree(root *TreeNode) int {
	ans := 0
	var dfs func(node *TreeNode) int
	dfs = func(node *TreeNode) int {
		if node == nil {
			return 0
		}
		l, r := dfs(node.Left), dfs(node.Right)
		ans = max(ans, l+r+1)
		return max(l, r) + 1
	}
	return max(ans, dfs(root)) - 1
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

- 如果扩展到多叉树呢，看看这道题：https://leetcode.cn/problems/tree-diameter/ （可惜是个会员题）



#### 泡椒风爪

- [2538. Difference Between Maximum and Minimum Price Sum](https://leetcode.cn/problems/difference-between-maximum-and-minimum-price-sum/)

这是一道2397分的周赛题Q4，当时没做出来，虽然我判断出了实质，但是很不巧，不会**树的直径**的技巧，只能打出GG，无奈吞下有一个三题周末。

如今，我已经变强了（不是），尝试再次分析这个问题。题目需要我找出一个价值最大和价值最小的差值，这个差值同样要最大：

1. 价值最小，那么就绝对是一个点，这个节点也有一定的特殊性，这个我们以后提到；
2. 价值最大，那么绝对是一个叶子节点到另一个叶子节点，否则必然存在至少一条路径，将其邻接节点添加进来，绝对比当前选的路径来的高价值，这也就给上一个思考带来了更加特殊的限定：价值最小会出现在叶子节点上；

我们归纳一下，接下来是要找树的直径，铁定是一条从叶子节点到叶子节点的通路，但是所求的结果，将会减去某一个通路端点的值。

接下来，我们就要面对编程技巧了。我们假定我们现在递归到`cur`这个点，那么，我们需要知道除开来时的路`fa`以外，最大的路径和次大的路径值。一开始考虑维护这个数据结构，但是灵神给了我另一个思路：

- 我们假设最大值为$m_1$，次大值为$m_2$，那么我其实只要维护一个变量`max`，就行了，但是要额外对`ans`进行更新，更新的策略是：`ans=max(ans, max(带节点的当前最大值+不带节点的当前值, 不带节点的当前最大值+带节点的当前值))`。这是由于如果$m_1$在$m_2$之前出现，那么`带节点的当前最大值`可定是最大值，那必然会出现$m_1+m_2$的情况，反之亦然，这下轮到$m_2$先出现。

总而言之，代码就和直白了：

##### **代码**

```go
func maxOutput(n int, edges [][]int, price []int) int64 {
	// build tree
	list := make([][]int, n)
	for _, edge := range edges {
		list[edge[0]] = append(list[edge[0]], edge[1])
		list[edge[1]] = append(list[edge[1]], edge[0])
	}
	ans := int64(0)

	var dfs func(cur, fa int) (int64, int64)
	dfs = func(cur, fa int) (int64, int64) {
		p := int64(price[cur])
		maxS1, maxS2 := p, int64(0)
		for _, next := range list[cur] {
			if next != fa {
				s1, s2 := dfs(next, cur)
				ans = max(ans, max(maxS1+s2, maxS2+s1))
				maxS1 = max(maxS1, s1+p)
				maxS2 = max(maxS2, s2+p)
			}
		}
		return maxS1, maxS2
	}
	dfs(0, -1)
	return ans
}

func max(a, b int64) int64 {
	if a > b {
		return a
	}
	return b
}
```

- 时间复杂度：$O(n)$。
- 空间复杂度：$O(n)$。



#### 虎皮风爪

- [124. Binary Tree Maximum Path Sum](https://leetcode.cn/problems/binary-tree-maximum-path-sum/)

作为餐前水果的衍生题，我们观察到节点的数值竟然有负数，这就意味着，结果可能只有一个节点，并不再是越长越好了，这样也毫不影响代码的整体结构，依然是一个递归回溯的过程：

- 在递归中计算当前节点作为根节点的最大路径，这个依旧不变为：`dfs(node.Left)+dfs(node.Right)`；
- 返回值这里变化了一下，以前因为非负数的原因，必然是返回`max(dfs(node.Left),dfs(node.Right))+node.Val`，但是这里不太一样了，我可以不选择叶子节点到叶子节点，我甚至可以选择非叶节点到非叶节点，所以多一个与`0`的`max`判断。

##### 代码

```go
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func maxPathSum(root *TreeNode) int {
	ans := math.MinInt
	var dfs func(node *TreeNode) int
	dfs = func(node *TreeNode) int {
		if node == nil {
			return 0
		}
		l, r := dfs(node.Left), dfs(node.Right)
		sum := l + r + node.Val
		ans = max(ans, sum)
		return max(l+node.Val, max(r+node.Val, 0))
	}
	dfs(root)
	return ans
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```



####  大盘鸡

- [2246. Longest Path With Different Adjacent Characters](https://leetcode.cn/problems/longest-path-with-different-adjacent-characters/)

这部分是鸡的专场（不是）。这依然是个树型结构的遍历回溯过程，但是判断上有点小**TIPS**，更新`ans`和`dfs`遍历并不统一：

```go
func longestPath(parent []int, s string) int {
	n := len(parent)
	list := make([][]int, n)
	for i := 0; i < n; i++ {
		if parent[i] != -1 {
			list[parent[i]] = append(list[parent[i]], i)
		}
	}

	ans := 0

	var dfs func(cur int) int
	dfs = func(cur int) int {
		maxS := 0
		for _, next := range list[cur] {
			nextS := dfs(next)
			if s[next] != s[cur] {
				ans = max(ans, maxS+1+nextS)
				maxS = max(maxS, nextS)
			}
		}
		return maxS + 1
	}
	return max(ans, dfs(0))
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```



#### 龙虾三吃

- [1617. Count Subtrees With Max Distance Between Cities](https://leetcode.cn/problems/count-subtrees-with-max-distance-between-cities/)

上点强度，来到2308分的题，虽然比起2397的题少了不少分，但是个人觉得还是比那道题难点，这道难在结合了两个思路，那道则难在如何抽象和返回两个值得操作。所以，我们继续紧跟灵神脚步，分析一下这道每日一题（恼）。

首先看一下数据量：$2 \le n \le 15$，数据量不大，而且需要枚举子集，那么显然可以肆无忌惮地fan out了，进行一波**DFS**。接着，要算子集的最大距离，那就回归到原有的思路上了，继续**DFS**，归纳来看，就是**DFS**套一个**DFS**：

##### 代码

```go
func countSubgraphsForEachDiameter(n int, edges [][]int) []int {
	list := make([][]int, n+1)
	for _, edge := range edges {
		list[edge[0]] = append(list[edge[0]], edge[1])
		list[edge[1]] = append(list[edge[1]], edge[0])
	}
	inSet := [16]bool{}
	ans := make([]int, n-1)
	vis, d := [16]bool{}, 0

	var dfs func(cur int) int
	dfs = func(cur int) int {
		res := 0
		vis[cur] = true
		for _, next := range list[cur] {
			if !vis[next] && inSet[next] {
				tmp := dfs(next)
				d = max(d, res+tmp+1)
				res = max(res, tmp+1)
			}
		}
		return res
	}

	var subF func(i int)
	subF = func(i int) {
		if i == n+1 {
			for cur, existed := range inSet {
				if existed {
					vis, d = [16]bool{}, 0
					dfs(cur)
					break
				}
			}
			if d > 0 && vis == inSet {
				ans[d-1]++
			}
			return
		}
		// ignore
		subF(i + 1)

		// select
		inSet[i] = true
		subF(i + 1)
		inSet[i] = false
	}

	subF(1)
	return ans
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```



灵神同样也教了我们其他优化的做法，首先优化一下递归的过程，依然是从数据量上下手，我们发现数据量少得可怜，可以用二进制优化一下：

```go
func countSubgraphsForEachDiameter(n int, edges [][]int) []int {
    // 建树
    g := make([][]int, n)
    for _, e := range edges {
        x, y := e[0]-1, e[1]-1 // 编号改为从 0 开始
        g[x] = append(g[x], y)
        g[y] = append(g[y], x)
    }

    ans := make([]int, n-1)
    // 二进制枚举
    for mask := 3; mask < 1<<n; mask++ {
        if mask&(mask-1) == 0 { // 需要至少两个点
            continue
        }
        // 求树的直径
        vis, diameter := 0, 0
        var dfs func(int) int
        dfs = func(x int) (maxLen int) {
            vis |= 1 << x // 标记 x 访问过
            for _, y := range g[x] {
                if vis>>y&1 == 0 && mask>>y&1 > 0 { // y 没有访问过且在 mask 中
                    ml := dfs(y) + 1
                    diameter = max(diameter, maxLen+ml)
                    maxLen = max(maxLen, ml)
                }
            }
            return
        }
        dfs(bits.TrailingZeros(uint(mask))) // 从一个在 mask 中的点开始递归
        if vis == mask {
            ans[diameter-1]++
        }
    }
    return ans
}
```



最后一个，看不懂了（放弃），直接上连接：

- [【图解】回溯/二进制枚举/O(n^3)枚举直径端点（Python/Java/C++/Go） - 统计子树中城市之间最大距离 - 力扣（LeetCode）](https://leetcode.cn/problems/count-subtrees-with-max-distance-between-cities/solution/tu-jie-on3-mei-ju-zhi-jing-duan-dian-che-am2n/)

```go
func countSubgraphsForEachDiameter(n int, edges [][]int) []int {
    // 建树
    g := make([][]int, n)
    for _, e := range edges {
        x, y := e[0]-1, e[1]-1 // 编号改为从 0 开始
        g[x] = append(g[x], y)
        g[y] = append(g[y], x)
    }

    // 计算树上任意两点的距离
    dis := make([][]int, n)
    for i := range dis {
        // 计算 i 到其余点的距离
        dis[i] = make([]int, n)
        var dfs func(int, int)
        dfs = func(x, fa int) {
            for _, y := range g[x] {
                if y != fa {
                    dis[i][y] = dis[i][x] + 1 // 自顶向下
                    dfs(y, x)
                }
            }
        }
        dfs(i, -1)
    }

    ans := make([]int, n-1)
    for i, di := range dis {
        for j := i + 1; j < n; j++ {
            dj := dis[j]
            d := di[j]
            var dfs func(int, int) int
            dfs = func(x, fa int) int {
                // 能递归到这，说明 x 可以选
                cnt := 1 // 选 x
                for _, y := range g[x] {
                    if y != fa &&
                       (di[y] < d || di[y] == d && y > j) &&
                       (dj[y] < d || dj[y] == d && y > i) { // 满足这些条件就可以选
                        cnt *= dfs(y, x) // 每棵子树互相独立，采用乘法原理
                    }
                }
                if di[x]+dj[x] > d { // x 是可选点
                    cnt++ // 不选 x
                }
                return cnt
            }
            ans[d-1] += dfs(i, -1)
        }
    }
    return ans
}
```

