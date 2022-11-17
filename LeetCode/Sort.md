## 排序（Sort）



#### 归并排序（Merge Sort）

- **稳定**的排序方法，时间复杂度**最优**、**最坏**和**平均**都是 $O(n\,log\,n)$ ，时间复杂度为 $O(n)$ ；

- 归并排序可以只使用 $O(1)$ 的辅助空间。

  ```go
  //  [l,r)
  func merge(l, r int) {
  	if r-l <= 1 {
  		return
  	}
  	mid := l + (l+r)>>1
  	merge(l, mid)
  	merge(mid, r)
  	for i, j, k := l, mid, l; k < r; k++ {
  		if j == r || (i < mid && a[i] <= a[j]) {
  			tmp[k] = a[i]
  			i++
  		} else {
  			tmp[k] = a[j]
  			j++
  		}
  	}
  	for i := l; i < r; i++ {
  		a[i] = tmp[i]
  	}
  }
  ```

- **典型例题**：[315. Count of Smaller Numbers After Self](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/)，**判断后序数列中有几个比当前数少**  ：

  - 通过归并排序部分有序的特性，每次会**合并前后两部分**，那么我只要在合并的时候关注后一部分移到前一部分的个数即可：

    ![merge-sort](images/merge-sort.svg)



#### 桶排序（Bucket Sort）

- 桶排序（`Bucket sort`）是排序算法的一种，适用于**待排序数据值域较大但分布比较均匀**的情况。一般情况下认为**桶排序**是一种**稳定**的排序算法。
- 桶排序按下列步骤进行：
  1. 设置一个定量的数组当作空桶；
  2. 遍历序列，并将元素一个个放到对应的桶中；
  3. 对每个不是空的桶进行排序；
  4. 从不是空的桶里把元素再放回原来的序列中。
- 桶排序的平均时间复杂度为 $O(n+n^2/k+k)$ （将值域平均分成 $k$ 块 + 排序 + 重新合并元素），当 $k \approx n$ 时为 $O(n)$ 。



#### 其他排序

##### 排序的最小交换次数

考虑一个数组，里面的元素可能**无序**也有可能**有序**，现在需要将该数组通过**两两交换**变成有序数组，请给出交换的**最小次数**。

虽然说是最小，但是只要通过规定操作就能判断，不妨考虑如下数组：`[5,3,1,2,4]`，其有序应为：`[1,2,3,4,5]`，我们考虑如下次数的交换即可：

1. 第一个数字应为 `1` ，那么交换过来：`[1,3,5,2,4]`；
2. 第二个数字应为 `2` ，那么交换过来：`[1,2,5,3,4]`；
3. 第三个数字应为 `3` ，那么交换过来：`[1,2,3,5,4]`；
4. 第四个数字应为 `4` ，那么交换过来：`[1,2,3,4,5]`；

**完成！**

同时我们不难发现，从**有序**到**无序**的过程就是上述过程的逆过程，也可以用相似的理论来解释。

###### 代码

```go
type node struct {
	val, idx int
}

func calTimes(nums []*node) int {
	if len(nums) == 1 {
		return 0
	}
	times := 0
	for i := 0; i < len(nums); i++ {
		nums[i].idx = i
	}
	sort.Slice(nums, func(i, j int) bool {
		return nums[i].val < nums[j].val
	})
	for i := 0; i < len(nums); {
		if j := nums[i].idx; j != i {
			nums[i], nums[j] = nums[j], nums[i]
			times++
		} else {
			i++
		}
	}
	return times
}
```



#### 排序例题

| ID   | LeetCode 题号                                                | 描述                                                         | 思路                                                        |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| 1    | [315. Count of Smaller Numbers After Self](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/) | 判断后序数列中有几个比当前数少                               | 归并排序中计算前半个数组中数据在排序中的偏移                |
| 2    | [剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/) | 前面一个数字大于后面的数字即组成一个逆序对，求数组中逆序对个数 | 转移为 `ID-1`，归并排序中计算前半个数组中数据在排序中的偏移 |
| 3    | [973. K Closest Points to Origin](https://leetcode.cn/problems/k-closest-points-to-origin/) | 离原点最近的 `K` 个点                                        | 暴力排序                                                    |
| 4    | [6235. Minimum Number of Operations to Sort a Binary Tree by Level](https://leetcode.cn/problems/minimum-number-of-operations-to-sort-a-binary-tree-by-level/) | 二叉树每层调整为升序，求最小调整次数                         | 数组有序最小调整次数 + `BFS` 层序遍历                       |

