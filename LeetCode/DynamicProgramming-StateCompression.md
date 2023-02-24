## 状态压缩DP专题——博弈



状态压缩是DP中一种常见的情况，通常与遍历有关，例如**DFS**方法。因此，出现了一系列的算法题，博弈问题就在其中，我们称之为，**博弈类DP**，此类DP有一个先决条件，就是Alice和Bob都是冷酷的敌人。

- 参考：https://leetcode.cn/problems/stone-game-ii/solution/jiao-ni-yi-bu-bu-si-kao-dong-tai-gui-hua-jjax/



### 餐前水果（Rank 1435）

- 题目：[1025. Divisor Game](https://leetcode.cn/problems/divisor-game/)

一开始我以为是三进制问题，后来才发现有一个先决条件，就是大家都是风灵月影宗的，能看到未来。所以，无外乎两种情况，Alice先手赢（Bob先手输），Alice先手输（Bob先手赢）。所以，我只需要存储Alice先手的输赢情况。我们列出转移方程：
$$
dp[i]=dp[i-j_1] \lor dp[i-j_2] \lor \cdots \lor dp[i-j_n] \qquad i \; mod \; j_t=0
$$
只要列出方程，万事大吉。



### 凉拌黄瓜（Rank 1590）

- 题目：[877. Stone Game](https://leetcode.cn/problems/stone-game/)

我们先看一下题目，Alice和Bob每次只能取队头和队尾的元素，意味着剩下的元素都是连续的，那么，我们列一下转移方程：
$$
dp[l][r]=\max{
	\begin{cases}
		dp[l+1][r-1]+piles[l]-piles[r] \\
        dp[l+1][r-1]+piles[r]-piles[l] \\
        dp[l+2][r]+piles[l]-piles[l+1] \\
        dp[l][r-2]+piles[l]-piles[r-1] \\
	\end{cases}
}
$$
又AC一道！

> 但是，我们考虑一下这道题的构成，我们可以发现，其实，Alice永远是赢家，所以：
>
> ```go
> return true
> ```
>
> 回头看一下我们的餐前水果，发现他也可以用数学推导出一个有趣的事实：偶数Alice必定先手赢，很好：
>
> ```go
> return n%2 == 0
> ```
>
> 我是🦈🖊。



### 土豆牛肉（Rank  2026）

- 题目：[1406. Stone Game III](https://leetcode.cn/problems/stone-game-iii/)

Alice和Bob又来玩游戏了，这次需要选择取几堆，但是好在数量是固定的1、2、3，坏消息是，神特么有负数。经过分析，得出一个递推的思路，假设我是Alice，我不知道取1、2、3堆哪次好，那我就计算Bob的，只要Bob取的得分少，那自然而然我们得分就高。又由于两人都是开挂玩家，自然而然先手从下标`i`开始取，只会有一种最优结果，所以我们可以缓存这种最优结果。只要判断取完1、2、3堆后，Bob能取得的最少分数就行。

由于每次取的次数是已知的，我们不妨从数组尾部开始往前推导。`dp[i]`表示从当前位置（包含）开始往后取，最多能取得多少分。那就有：
$$
dp[i]=stoneValue[i-1+1]+\cdots+stoneValue[i-1+token]+\{NextRound\}
$$
`token`很好取，判断一下$dp[i+token]\quad token \in \{1,2,3\}$哪个最小，那么，下一轮的计数如何获得呢。我们用后缀和$S$优化，得到递推方程：
$$
dp[i]=S[i]-dp[i+token]
$$
很直白，当前下标$i$往后的和为$S[i]$，对手最多能取走$dp[i+token]$，那么自然而然剩下的就是我的了。

最后，处理一下边界情况，等到达边界，即剩余个数小于等于3个，也肯定多于1个，所以我们需要两个$dp$的特殊位置，里面填上什么数字呢？分析一下，如果末尾剩1个，无论正负你都得吃下，要不影响公式，则$dp[i+token]$为0；剩余两个呢，两个正，都拿，两个负，拿一个，正负负正两个都拿，表现出的就是和0做对比，还是契合公式。所以，完备了。A！C！



### 北京烤鸭（Rank 2034）

- 题目：[1140. Stone Game II](https://leetcode.cn/problems/stone-game-ii/)

思路过程看**Contest**的一篇文章。



375 - ?

464 - ?

486 - ?

