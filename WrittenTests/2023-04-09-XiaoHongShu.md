## 小红书笔试 2023-04-09场



### 第一题

给定一棵树$1 \dots n$（根节点未知），任意删去一条边形成了两棵树，求这两棵树节点数量之差的最小值，若有多个，统计同为最小值时能删边的方案数。

首先输入$n$表示节点数，随后$n-1$行每行分别输入$s$和$t$，表示节点$s$的父节点为$t$。

- $1 \leq n \le 10^5$

**样例输入**

```shell
3	# n
2 1	# s t
3 1 # s t
```

**样例输出**

```shell
1 2	# min cnt
```



#### 思路

这次输入是比较特殊，只告诉了你父节点，这很容易往并查集上想，干扰思路。但是稍稍分析一下，还是典型的一条边，**直接加入邻接表**，只是这次需要去判断哪个是根节点。这次没有显式告诉，但是由于只接受了$n-1$个输入，那么根节点`root`只需要再次遍历一遍父节点数组找到`fa[root]==0`。

接下来处理核心逻辑。我们观察到这道题，任意断一条边：

![20230409-xiaohongshu-1-example01](images/20230409-xiaohongshu-1-example01.svg)

那么，蓝色的子树的节点个数为3，绿色为4，那么所求的差为1。不难看出，假设一棵子树的节点个数为$k$，那么这个差值为$|(n-k)-k|$。同时我们也知道，断开的边“下方”的那棵子树不涉及遍历回根然后再向下的遍历过程，比较好计算。

最后，只需要一个`DFS`来完成这棵子树节点个数的计算，一遍向下再回溯，很容易得出。同时，得出该数据后，立即更新结果，如果比最小值小，直接更新；若和最小值相等，则计数加一。



#### 代码

```go
package main

import "fmt"

func main() {
	var s, t int
	var n int
	fmt.Scan(&n)

	fa := make([]int, n+1)
	list := make([][]int, n+1)

	for i := 1; i < n; i++ {
		fmt.Scan(&s, &t)
		list[s] = append(list[s], t)
		list[t] = append(list[t], s)
		fa[s] = t
	}

	root := 0
	for i := 1; i <= n; i++ {
		if fa[i] == i || fa[i] == 0 {
			root = i
			break
		}
	}

	min, cnt := n, 0
	var dfs func(father, cur int) int
	dfs = func(father, cur int) int {
		res := 1
		for _, next := range list[cur] {
			if next != father {
				res += dfs(cur, next)
			}
		}
		if a := abs(n-res, res); a < min {
			min = a
			cnt = 1
		} else if a == min {
			cnt++
		}
		return res
	}

	dfs(fa[root], root)

	fmt.Printf("%d %d\n", min, cnt)
}

func abs(a, b int) int {
	if a > b {
		return a - b
	}
	return b - a
}
```



#### 复杂度

- 时间复杂度：$O(n)$
- 空间复杂度：$O(n)$



### 第二题

提供$n$种溶液，每种溶液体积为$x_i$，价值为$y_i$，配置溶液，要求体积恰好为$C$，规则如下：

1. 每次选择两种溶液，溶液配置体积相加，溶液价值相加；
2. 若两种溶液体积相等，则触发奖励，溶液价值相加的同时增加$X$；
3. 溶液无限量使用；

- $1 \le n,x_i,y_i,X,C \le 10^3$

**样例输入**

```shell
3 4 16	# n X C
5 3 4	# xi
2 4 1	# yi
```

**样例输出**

```shell
29	# max value
```



#### 思路

这是一个典型得不能再典型的可以无限取的**DP背包问题**，但是我们还能以另一种思路来考虑：

1. 首先我需要配置$C_i$体积的液体，则需要两个体积相加为$C_i$的液体混合，遍历这些液体的同时使用最大值去更新：
   $$
   dp[C_i] = 
   \begin{cases}
   	2 \times dp[C_i/2]+X \qquad 2 \times (C_{i}/2) =C_i \\
       dp[C_{ii}]+dp[C_{ij}] \qquad C_{ii}+C_{ij} =C_i \\
   \end{cases}
   $$

2. 其实写出这个转移方程基本上就算结束了，有个小优化点就是$C_{ii}$并不需要遍历$[1,C_i-1]$，只需要遍历一般就行了，即$[1,C_i/2]$。

3. 小技巧：可以直接用输入的$x_i$和$y_i$更新`DP`数组。



#### 代码

```go
package main

import "fmt"

func main() {
	var n, X, C int
	fmt.Scan(&n, &X, &C)
	dp := make([]int, C+1)

	Xs := make([]int, n)
	var x, y int
	for i := 0; i < n; i++ {
		fmt.Scan(&x)
		Xs[i] = x
	}
	for i := 0; i < n; i++ {
		fmt.Scan(&y)
		dp[Xs[i]] = max(dp[Xs[i]], y)
	}

	for i := 1; i <= C; i++ {
		for j := i - 1; j >= 1; j-- {
			if dp[j] == 0 || dp[i-j] == 0 {
				continue
			}
			if j*2 == i {
				dp[i] = max(dp[i], dp[i-j]+dp[j]+X)
			} else {
				dp[i] = max(dp[i], dp[i-j]+dp[j])
			}
		}
	}

	fmt.Println(dp[C])
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```



#### 复杂度

- 时间复杂度：$O(n^2)$
- 空间复杂度：$O(n)$



### 第三题

摸球游戏，一个袋子里有$n$个球，球上有一个数字，分别为红蓝球，有如下规定：

1. 开始时为空口袋，每一次操作可以取球，放球，和查询：
   - `0` —— 查询当前袋子里所有球上积分之和；
   - `-x`——从袋子里取出编号为`x`的球；
   - `x` —— 把编号为`x`球放入袋子里；
2. 每次操作都有一个实践戳，每次时间增长，都会带来以下效果：
   - 袋子里红球上的数字`+1`；
   - 袋子里篮球上的数字`-1`；

**输入样例**

```shell
3						# n
-5 4 6					# 球上编号
RBR						# 球颜色
9						# m, 操作次数
1 2 3 4 5  6 8  13 14	# 操作时间戳
1 3 0 2 -3 0 -1 0  -2	# 操作数
```

样例输出

```shell
3 4 2 -5				# 查询次数 (每次查询的积分)
```



#### 思路

这道题的主要思路是**延迟更新**，延迟更新怎么做呢，先看一下正常更新（以红球为例）：

![20230409-xiaohongshu-3-example01](images/20230409-xiaohongshu-3-example01.svg)

加入一个红球：

![20230409-xiaohongshu-3-example02](images/20230409-xiaohongshu-3-example02.svg)

拿出一个红球：

![20230409-xiaohongshu-3-example03](images/20230409-xiaohongshu-3-example03.svg)

这是正常更新，但是明显会造成$O(nm)$的时间复杂度，显然不能过。那么考虑一下延时，怎么考虑呢？

1. 袋子里球数是固定的，每秒增长值是固定的，全体球都会增长；
2. 袋子外面的球就可以不管；
3. 每次都带有时间戳，也就是说，一个球增长的积分是可以计算出的；

那么，我们现在不急着直接更新红球，而是加个虚拟的绿球：

![20230409-xiaohongshu-3-example12](../../../Drawios/Exam/20230409-xiaohongshu-3-example12.svg)

总积分用额外的`RedScore`去维护，若此时红球球数`RedSize`，则`RedScore+=RedSize*(timestamp-last_timestamp)`；

![20230409-xiaohongshu-3-example13](../../../Drawios/Exam/20230409-xiaohongshu-3-example13.svg)

当一个球被取出来，此时我再做真正的更新：红球数据更新，`RedSize`和`RedScore`更新。

我们这里有绿球来表示虚拟的增长，那么代码中呢？加个进袋的时间戳就行了。想想，是不是很美妙？😀



#### 代码

```go
package main

import "fmt"

type node struct {
	putInTime int64
	score     int64
}

func main() {
	// line one
	var n int
	fmt.Scan(&n)
	nodes := make([]*node, n+1)

	// line two
	var num int64
	for i := 1; i <= n; i++ {
		fmt.Scan(&num)
		nodes[i] = &node{score: num, putInTime: -1}
	}

	// line three
	var colors string
	fmt.Scan(&colors)

	// line four
	var m int64
	fmt.Scan(&m)

	// line five
	times := make([]int64, m)
	for i := int64(0); i < m; i++ {
		fmt.Scan(&times[i])
	}

	cnt := int64(0)
	var op int

	redSize := int64(0)
	redScore := int64(0)
	blueSize := int64(0)
	blueScore := int64(0)

	// line six
	ans := make([]int64, 0)
	last := int64(0)
	for i := int64(0); i < m; i++ {
		time := times[i]
		redScore += (time - last) * redSize
		blueScore -= (time - last) * blueSize
		last = time

		fmt.Scan(&op)
		if op == 0 {
			ans = append(ans, redScore+blueScore)
			cnt++
		} else if op > 0 {
			// put in
			if colors[op-1] == 'R' {
				// red
				redSize++
				redScore += nodes[op].score
				nodes[op].putInTime = time
			} else {
				// blue
				blueSize++
				blueScore += nodes[op].score
				nodes[op].putInTime = time
			}
		} else {
			op = -op
			// take out
			if colors[op-1] == 'R' {
				// red
				redSize--
				nodes[op].score += time - nodes[op].putInTime
				redScore -= nodes[op].score
			} else {
				// blue
				blueSize--
				nodes[op].score -= time - nodes[op].putInTime
				blueScore -= nodes[op].score
			}
		}
	}

	fmt.Printf("%d", cnt)
	for _, an := range ans {
		fmt.Printf(" %d", an)
	}
	fmt.Println()
}
```



#### 复杂度

- 时间复杂度：$O(\max(n,m))$
- 空间复杂度：$O(n)$